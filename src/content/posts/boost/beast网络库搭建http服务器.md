---
title: beast网络库搭建http服务器
published: 2026-05-16
# description: 
# image: ./cover.jpg
tags: [boost]
category: boost
draft: false
# slug: how-to-use-firefly
---
---

## 简介
前面的几篇文章已经介绍了如何使用asio搭建高并发的tcp服务器，以及http服务器。但是纯手写http服务器太麻烦了，有网络库beast已经帮我们实现了。这一期讲讲如何使用beast实现一个http服务器。

## 连接类
我们先实现http_server函数

```cpp
void http_server(tcp::acceptor& acceptor, tcp::socket& socket)
{
    acceptor.async_accept(socket,
        [&](beast::error_code ec)
        {
            if (!ec)
                std::make_shared<http_connection>(std::move(socket))->start();
            http_server(acceptor, socket);
        });
}
```

http_server中添加了异步接收连接的逻辑，当有新的连接到来时创建`http_connection`,然后启动服务，新连接监听对端数据。接下来http_server继续监听对端的新连接。  
连接类`http_connection`里实现了start函数监听对端数据

```cpp
void start()
{
    read_request();
    check_deadline();
}
```

处理读请求,将读到的数据存储再成员变量`request_`中，然后调用`process_request`处理请求

```cpp
void read_request()
{
    auto self = shared_from_this();

    http::async_read(
        socket_,
        buffer_,
        request_,
        [self](beast::error_code ec,
                std::size_t bytes_transferred)
        {
            boost::ignore_unused(bytes_transferred);
            if (!ec)
                    self->process_request();
        });
}
```

`check_deadline`主要时用来检测超时，当超过一定时间后自动关闭连接，因为http请求时短链接

```cpp
    void
        check_deadline()
    {
        auto self = shared_from_this();

        deadline_.async_wait(
            [self](beast::error_code ec)
            {
                if (!ec)
                {
                    // Close socket to cancel any outstanding operation.
                    self->socket_.close(ec);
                }
            });
    }
```

`process_request`函数中区分请求的类型，进行不同类型的处理如post还是get请求

```cpp
void process_request()
{
    response_.version(request_.version());
    response_.keep_alive(false);

    switch (request_.method())
    {
        case http::verb::get:
            response_.result(http::status::ok);
            response_.set(http::field::server, "Beast");
            create_response();
            break;
        case http::verb::post:
            response_.result(http::status::ok);
            response_.set(http::field::server, "Beast");
            create_post_response();
            break;
        default:
            // We return responses indicating an error if
            // we do not recognize the request method.
            response_.result(http::status::bad_request);
            response_.set(http::field::content_type, "text/plain");
            beast::ostream(response_.body())
                << "Invalid request-method '"
                << std::string(request_.method_string())
                << "'";
            break;
        }

    write_response();
}
```

`create_response`函数中解析了不同的路由处理get请求

```cpp
    void
        create_response()
    {
        if (request_.target() == "/count")
        {
            response_.set(http::field::content_type, "text/html");
            beast::ostream(response_.body())
                << "<html>\n"
                << "<head><title>Request count</title></head>\n"
                << "<body>\n"
                << "<h1>Request count</h1>\n"
                << "<p>There have been "
                << my_program_state::request_count()
                << " requests so far.</p>\n"
                << "</body>\n"
                << "</html>\n";
        }
        else if (request_.target() == "/time")
        {
            response_.set(http::field::content_type, "text/html");
            beast::ostream(response_.body())
                << "<html>\n"
                << "<head><title>Current time</title></head>\n"
                << "<body>\n"
                << "<h1>Current time</h1>\n"
                << "<p>The current time is "
                << my_program_state::now()
                << " seconds since the epoch.</p>\n"
                << "</body>\n"
                << "</html>\n";
        }
        else
        {
            response_.result(http::status::not_found);
            response_.set(http::field::content_type, "text/plain");
            beast::ostream(response_.body()) << "File not found\r\n";
        }
    }
```

`create_post_response`处理了post请求中的一部分路由

```cpp
    void create_post_response() {
        if (request_.target() == "/email")
        {
            auto& body = this->request_.body();
            auto body_str = boost::beast::buffers_to_string(body.data());
            std::cout << "receive body is " << body_str << std::endl;
            this->response_.set(http::field::content_type, "text/json");
            Json::Value root;
            Json::Reader reader;
            Json::Value src_root;
            bool parse_success = reader.parse(body_str, src_root);
            if (!parse_success) {
                std::cout << "Failed to parse JSON data!" << std::endl;
                root["error"] = 1001;
                std::string jsonstr = root.toStyledString();
                beast::ostream(this->response_.body()) << jsonstr;
                return ;
            }

            auto email = src_root["email"].asString();
            std::cout << "email is " << email << std::endl;

            root["error"] = 0;
            root["email"] = src_root["email"];
            root["msg"] = "recevie email post success";
            std::string jsonstr = root.toStyledString();
            beast::ostream(this->response_.body()) << jsonstr;
        }
        else
        {
            response_.result(http::status::not_found);
            response_.set(http::field::content_type, "text/plain");
            beast::ostream(response_.body()) << "File not found\r\n";
        }
    }
```

`write_response`发送请求

```cpp
void write_response()
{
    auto self = shared_from_this();

    response_.content_length(response_.body().size());

    http::async_write(
        socket_,
        response_,
        [self](beast::error_code ec, std::size_t)
        {
            self->socket_.shutdown(tcp::socket::shutdown_send, ec);
            self->deadline_.cancel();
    });
}
```
