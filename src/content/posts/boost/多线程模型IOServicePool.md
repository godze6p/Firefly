---
title: 多线程模型IOServicePool
published: 2026-05-12
# description: 
# image: ./cover.jpg
tags: [boost]
category: boost
draft: false
# slug: how-to-use-firefly
---
---

## 简介
前面的设计，我们对asio的使用都是单线程模式，为了提升网络io并发处理的效率，这一次我们设计多线程模式下asio的使用方式。总体来说asio有两个多线程模型，第一个是启动多个线程，每个线程管理一个iocontext。第二种是只启动一个iocontext，被多个线程共享，后面的文章会对比两个模式的区别，这里先介绍第一种模式，多个线程，每个线程管理独立的iocontext服务。



## IOServicePool多线程模式特点

1   每一个io_context跑在不同的线程里，所以同一个socket会被注册在同一个io_context里，它的回调函数也会被单独的一个线程回调，那么对于同一个socket，他的回调函数每次触发都是在同一个线程里，就不会有线程安全问题，网络io层面上的并发是线程安全的。

2   但是对于不同的socket，回调函数的触发可能是同一个线程(两个socket被分配到同一个io_context)，也可能不是同一个线程(两个socket被分配到不同的io_context里)。所以如果两个socket对应的上层逻辑处理，如果有交互或者访问共享区，会存在线程安全问题。比如socket1代表玩家1，socket2代表玩家2，玩家1和玩家2在逻辑层存在交互，比如两个玩家都在做工会任务，他们属于同一个工会，工会积分的增加就是共享区的数据，需要保证线程安全。可以通过加锁或者逻辑队列的方式解决安全问题，我们目前采取了后者。

3   多线程相比单线程，极大的提高了并发能力，因为单线程仅有一个io_context服务用来监听读写事件，就绪后回调函数在一个线程里串行调用, 如果一个回调函数的调用时间较长肯定会影响后续的函数调用，毕竟是穿行调用。而采用多线程方式，可以在一定程度上减少前一个逻辑调用影响下一个调用的情况，比如两个socket被部署到不同的iocontext上，但是当两个socket部署到同一个iocontext上时仍然存在调用时间影响的问题。不过我们已经通过逻辑队列的方式将网络线程和逻辑线程解耦合了，不会出现前一个调用时间影响下一个回调触发的问题。

## IOServicePool实现
IOServicePool本质上是一个线程池，基本功能就是根据构造函数传入的数量创建n个线程和iocontext，然后每个线程跑一个iocontext，这样就可以并发处理不同iocontext读写事件了。

IOServicePool的声明

```cpp
class AsioIOServicePool:public Singleton<AsioIOServicePool>
{
    friend Singleton<AsioIOServicePool>;
public:
    using IOService = boost::asio::io_context;
    using Work = boost::asio::io_context::work;
    using WorkPtr = std::unique_ptr<Work>;
    ~AsioIOServicePool();
    AsioIOServicePool(const AsioIOServicePool&) = delete;
    AsioIOServicePool& operator=(const AsioIOServicePool&) = delete;
    // 使用 round-robin 的方式返回一个 io_service
    boost::asio::io_context& GetIOService();
    void Stop();
private:
    AsioIOServicePool(std::size_t size = std::thread::hardware_concurrency());
    std::vector<IOService> _ioServices;
    std::vector<WorkPtr> _works;
    std::vector<std::thread> _threads;
    std::size_t   _nextIOService;
};
```

1   `_ioServices`是一个IOService的vector变量，用来存储初始化的多个IOService。

2   `WorkPtr`是`boost::asio::io_context::work`类型的unique指针。  
在实际使用中，我们通常会将一些异步操作提交给`io_context`进行处理，然后该操作会被异步执行，而不会立即返回结果。如果没有其他任务需要执行，那么`io_context`就会停止工作，导致所有正在进行的异步操作都被取消。这时，我们需要使用`boost::asio::io_context::work`对象来防止`io_context`停止工作。

`boost::asio::io_context::work`的作用是持有一个指向`io_context`的引用，并通过创建一个“工作”项来保证`io_context`不会停止工作，直到work对象被销毁或者调用`reset()`方法为止。当所有异步操作完成后，程序可以使用`work.reset()`方法来释放`io_context`，从而让其正常退出。

3   `_threads`是一个线程vector,管理我们开辟的所有线程。

4   `_nextIOService`是一个轮询索引，我们用最简单的轮询算法为每个新创建的连接分配io_context.

5   因为IOServicePool不允许被copy构造，所以我们将其拷贝构造和拷贝复制函数置为delete

接下来我们实现构造函数

```cpp
AsioIOServicePool::AsioIOServicePool(std::size_t size):_ioServices(size),
_works(size), _nextIOService(0){
    for (std::size_t i = 0; i < size; ++i) {
        _works[i] = std::unique_ptr<Work>(new Work(_ioServices[i]));
    }

    //遍历多个ioservice，创建多个线程，每个线程内部启动ioservice
    for (std::size_t i = 0; i < _ioServices.size(); ++i) {
        _threads.emplace_back([this, i]() {
            _ioServices[i].run();
            });
    }
}
```

_works是unique_ptr的vector类型，所以初始化时要么放在构造函数初始化列表里初始化，要么通过一个临时的`std::unique_ptr`右值初始化，我们采取的是第二种。

实现获取`io_context&`的函数

```cpp
boost::asio::io_context& AsioIOServicePool::GetIOService() {
    auto& service = _ioServices[_nextIOService++];
    if (_nextIOService == _ioServices.size()) {
        _nextIOService = 0;
    }
    return service;
}
```

我们根据`_nextIOService`作为索引，轮询获取`io_context&`。

同样我们要实现Stop函数，控制`AsioIOServicePool`停止的行为。因为我们要保证每个线程安全退出后再让`AsioIOServicePool`停止。

```cpp
void AsioIOServicePool::Stop(){
    for (auto& work : _works) {
        work.reset();
    }

    for (auto& t : _threads) {
        t.join();
    }
}
```

其中`work.reset()`是让unique指针置空并释放，那么work的析构函数就会被调用，work被析构，其管理的io_service在没有事件监听时就会被释放。

## 优雅退出
IOServicePool多线程服务器退出时，需要捕获退出信号如SIGINT,SIGTERM等，将退出信号和一个iocontext绑定，当收到退出信号时，我们将IOServicePool停止，并且停止iocontext即可。

```cpp
int main()
{
    try {
        auto pool = AsioIOServicePool::GetInstance();
        boost::asio::io_context  io_context;
        boost::asio::signal_set signals(io_context, SIGINT, SIGTERM);
        signals.async_wait([&io_context,pool](auto, auto) {
            io_context.stop();
            pool->Stop();
            });
        CServer s(io_context, 10086);
        io_context.run();
    }
    catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << endl;
    }

}
```

上例中，我们将一个iocontext绑定给Server，当收到退出消息后停止这个iocontext，并且停止IOServicePool。

