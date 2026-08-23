---
title: asio处理粘包的简易方式
published: 2026-05-09
# description: 
# image: ./cover.jpg
tags: [boost]
category: boost
draft: false
# slug: how-to-use-firefly
---
---

## 简易方式
之前我们介绍了通过async_read_some函数监听读事件，并且绑定了读事件的回调函数HandleRead

```cpp
_socket.async_read_some(boost::asio::buffer(_data, MAX_LENGTH), std::bind(&CSession::HandleRead, this, 
std::placeholders::_1, std::placeholders::_2, SharedSelf()));
```

async_read_some 这个函数的特点是只要对端发数据，服务器接收到数据，即使没有收全对端发送的数据也会触发HandleRead函数，所以我们会在HandleRead回调函数里判断接收的字节数，接收的数据可能不满足头部长度，可能大于头部长度但小于消息体的长度，可能大于消息体的长度，还可能大于多个消息体的长度，所以要切包等，这些逻辑写起来很复杂，所以我们可以通过读取指定字节数，直到读完这些字节才触发回调函数，那么可以采用async_read函数，这个函数指定读取指定字节数，只有完全读完才会触发回调函数。

## 获取头部数据
我们可以读取指定的头部长度，大小为HEAD_LENGTH字节数，只有读完HEAD_LENGTH字节才触发HandleReadHead函数

```cpp
void CSession::Start(){
    _recv_head_node->Clear();
    boost::asio::async_read(_socket, boost::asio::buffer(_recv_head_node->_data, HEAD_LENGTH), std::bind(&CSession::HandleReadHead, this, 
        std::placeholders::_1, std::placeholders::_2, SharedSelf()));
}
```

这样我们可以直接在HandleReadHead函数内处理头部信息

```cpp
void CSession::HandleReadHead(const boost::system::error_code& error, size_t  bytes_transferred, std::shared_ptr<CSession> shared_self) {
    if (!error) {
        if (bytes_transferred < HEAD_LENGTH) {
            cout << "read head lenth error";
            Close();
            _server->ClearSession(_uuid);
            return;
        }

        //头部接收完，解析头部
        short data_len = 0;
        memcpy(&data_len, _recv_head_node->_data, HEAD_LENGTH);
        cout << "data_len is " << data_len << endl;
        //此处省略字节序转换
        // ...
        //头部长度非法
        if (data_len > MAX_LENGTH) {
            std::cout << "invalid data length is " << data_len << endl;
            _server->ClearSession(_uuid);
            return;
        }

        _recv_msg_node= make_shared<MsgNode>(data_len);
        boost::asio::async_read(_socket, boost::asio::buffer(_recv_msg_node->_data, _recv_msg_node->_total_len), 
            std::bind(&CSession::HandleReadMsg, this,
            std::placeholders::_1, std::placeholders::_2, SharedSelf()));
    }
    else {
        std::cout << "handle read failed, error is " << error.what() << endl;
        Close();
        _server->ClearSession(_uuid);
    }
}
```

接下来根据头部内存储的消息体长度，获取指定长度的消息体数据，所以再次调用async_read，指定读取`_recv_msg_node->_total_len`长度，然后触发HandleReadMsg函数

## 获取消息体
HandleReadMsg函数内解析消息体，解析完成后打印收到的消息，接下来继续监听读事件，监听读取指定头部大小字节，触发HandleReadHead函数， 然后再在HandleReadHead内继续监听读事件，获取消息体长度数据后触发HandleReadMsg函数，从而达到循环监听的目的。

```cpp
void CSession::HandleReadMsg(const boost::system::error_code& error, size_t  bytes_transferred,
    std::shared_ptr<CSession> shared_self) {
    if (!error) {
        PrintRecvData(_data, bytes_transferred);
        std::chrono::milliseconds dura(2000);
        std::this_thread::sleep_for(dura);
        _recv_msg_node->_data[_recv_msg_node->_total_len] = '\0';
        cout << "receive data is " << _recv_msg_node->_data << endl;
        Send(_recv_msg_node->_data, _recv_msg_node->_total_len);
        //再次接收头部数据
        _recv_head_node->Clear();
        boost::asio::async_read(_socket, boost::asio::buffer(_recv_head_node->_data, HEAD_LENGTH),
            std::bind(&CSession::HandleReadHead, this, std::placeholders::_1, std::placeholders::_2,
                SharedSelf()));
    }
    else {
        cout << "handle read msg failed,  error is " << error.what() << endl;
        Close();
        _server->ClearSession(_uuid);
    }
}
```
