---
title: 利用asio协程调度实现并发服务器
published: 2026-05-14
# description: 
# image: ./cover.jpg
tags: [boost]
category: boost
draft: false
# slug: how-to-use-firefly
---
---

## 简介
利用协程实现并发程序有两个好处  
1   将回调函数改写为顺序调用，提高开发效率。  
2   协程调度比线程调度更轻量化，因为协程是运行在用户空间的，线程切换需要在用户空间和内核空间切换。

## 协程案例
asio官网提供了一个协程并发编程的案例，我们列举一下

```cpp

#include <boost/asio/co_spawn.hpp>
#include <boost/asio/detached.hpp>
#include <boost/asio/io_context.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <boost/asio/signal_set.hpp>
#include <boost/asio/write.hpp>
#include <cstdio>

using boost::asio::ip::tcp;
using boost::asio::awaitable;
using boost::asio::co_spawn;
using boost::asio::detached;
using boost::asio::use_awaitable;
namespace this_coro = boost::asio::this_coro;

#if defined(BOOST_ASIO_ENABLE_HANDLER_TRACKING)
# define use_awaitable \
  boost::asio::use_awaitable_t(__FILE__, __LINE__, __PRETTY_FUNCTION__)
#endif

awaitable<void> echo(tcp::socket socket)
{
    try
    {
        char data[1024];
        for (;;)
        {
            std::size_t n = co_await socket.async_read_some(boost::asio::buffer(data), use_awaitable);
            co_await async_write(socket, boost::asio::buffer(data, n), use_awaitable);
        }
    }
    catch (std::exception& e)
    {
        std::printf("echo Exception: %s\n", e.what());
    }
}

awaitable<void> listener()
{
    auto executor = co_await this_coro::executor;
    tcp::acceptor acceptor(executor, { tcp::v4(), 10086 });
    for (;;)
    {
        tcp::socket socket = co_await acceptor.async_accept(use_awaitable);
        co_spawn(executor, echo(std::move(socket)), detached);
    }
}

int main()
{
    try
    {
        boost::asio::io_context io_context(1);

        boost::asio::signal_set signals(io_context, SIGINT, SIGTERM);
        signals.async_wait([&](auto, auto) { io_context.stop(); });

        co_spawn(io_context, listener(), detached);

        io_context.run();
    }
    catch (std::exception& e)
    {
        std::printf("Exception: %s\n", e.what());
    }
}
```

1   我们用awaitable声明了一个函数，那么这个函数就变为可等待的函数了，比如`listener`被添加`awaitable<void>`之后，就可以被协程调用和等待了。 2   `co_spawn`表示启动一个协程，参数分别为调度器，执行的函数，以及启动方式, 比如我们启动了一个协程，deatched表示将协程对象分离出来，这种启动方式可以启动多个协程，他们都是独立的，如何调度取决于调度器，在用户的感知上更像是线程调度的模式，类似于并发运行，其实底层都是串行的。

```cpp
co_spawn(io_context, listener(), detached);
```

我们启动了一个协程，执行listener中的逻辑，listener内部co_await 等待 acceptor接收连接，如果没有连接到来则挂起协程。执行之后的`io_context.run()`逻辑。所以协程实际上是在一个线程中串行调度的，只是感知上像是并发而已。  
3   当acceptor接收到连接后，继续调用co_spawn启动一个协程，用来执行echo逻辑。echo逻辑里也是通过co_wait的方式接收和发送数据的，如果对端不发数据，执行echo的协程就会挂起，另一个协程启动，继续接收新的连接。当没有连接到来，接收新连接的协程挂起，如果所有协程都挂起，则等待新的就绪事件(对端发数据，或者新连接)到来唤醒。

## 改进服务器
我们可以利用协程改进服务器编码流程，用一个iocontext管理绑定acceptor用来接收新的连接，再用一个iocontext或以IOServicePool的方式管理连接的收发操作，在每个连接的接收数据时改为启动一个协程，通过顺序的方式读取收到的数据

```cpp
void CSession::Start() {
    auto shared_this = shared_from_this();
    //开启接收协程
    co_spawn(_io_context, [=]()->awaitable<void> {
        try {
            for (;!_b_close;) {
                _recv_head_node->Clear();
                std::size_t n = co_await boost::asio::async_read(_socket,
                    boost::asio::buffer(_recv_head_node->_data, HEAD_TOTAL_LEN),
                    use_awaitable);

                if (n == 0) {
                    std::cout << "receive peer closed" << endl;
                    Close();
                    _server->ClearSession(_uuid);
                    co_return;
                }

                //获取头部MSGID数据
                short msg_id = 0;
                memcpy(&msg_id, _recv_head_node->_data, HEAD_ID_LEN);
                //网络字节序转化为本地字节序
                msg_id = boost::asio::detail::socket_ops::network_to_host_short(msg_id);
                std::cout << "msg_id is " << msg_id << endl;
                //id非法
                if (msg_id > MAX_LENGTH) {
                    std::cout << "invalid msg_id is " << msg_id << endl;
                    _server->ClearSession(_uuid);
                    co_return;
                }
                short msg_len = 0;
                memcpy(&msg_len, _recv_head_node->_data + HEAD_ID_LEN, HEAD_DATA_LEN);
                //网络字节序转化为本地字节序
                msg_len = boost::asio::detail::socket_ops::network_to_host_short(msg_len);
                std::cout << "msg_len is " << msg_len << endl;
                //长度非法
                if (msg_len > MAX_LENGTH) {
                    std::cout << "invalid data length is " << msg_len << endl;
                    _server->ClearSession(_uuid);
                    co_return;
                }

                _recv_msg_node = make_shared<RecvNode>(msg_len, msg_id);
                //读出包体
                n = co_await boost::asio::async_read(_socket,
                    boost::asio::buffer(_recv_msg_node->_data, _recv_msg_node->_total_len), use_awaitable);

                if (n == 0) {
                    std::cout << "receive peer closed" << endl;
                    Close();
                    _server->ClearSession(_uuid);
                    co_return;
                }

                _recv_msg_node->_data[_recv_msg_node->_total_len] = '\0';
                cout << "receive data is " << _recv_msg_node->_data << endl;
                //投递给逻辑线程
                LogicSystem::GetInstance().PostMsgToQue(make_shared<LogicNode>(shared_from_this(), _recv_msg_node));
            }

        }
        catch (std::exception& e) {
            std::cout << "exception is " << e.what() << endl;
            Close();
            _server->ClearSession(_uuid);
        }
        }, detached);
    
}

```

其余的逻辑和之前大体相同，测试了一下在一个iocontext负责接收新连接，一个iocontext负责接收数据和发送数据的情况下，客户端创建100个连接，收发500次，总用时为55s  

