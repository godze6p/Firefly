---
title: asio实现http服务器
published: 2026-05-15
# description: 
# image: ./cover.jpg
tags: [boost]
category: boost
draft: false
# slug: how-to-use-firefly
---
## 简介
前文介绍了asio如何实现并发的长连接tcp服务器，今天介绍如何实现http服务器，在介绍实现http服务器之前，需要讲述下http报文头的格式，其实http报文头的格式就是为了避免我们之前提到的粘包现象，告诉服务器一个数据包的开始和结尾，并在包头里标识请求的类型如get或post等信息。

## HTTP包头信息
一个标准的HTTP报文头通常由请求头和响应头两部分组成。

### HTTP 请求头
HTTP请求头包括以下字段：

+ **Request-line**：包含用于描述请求类型、要访问的资源以及所使用的HTTP版本的信息。
+ **Host**：指定被请求资源的主机名或IP地址和端口号。
+ **Accept**：指定客户端能够接收的媒体类型列表，用逗号分隔，例如 text/plain, text/html。
+ **User-Agent**：客户端使用的浏览器类型和版本号，供服务器统计用户代理信息。
+ **Cookie**：如果请求中包含cookie信息，则通过这个字段将cookie信息发送给Web服务器。
+ **Connection**：表示是否需要持久连接（keep-alive）。

比如下面就是一个实际应用

```cpp
GET /index.html HTTP/1.1
Host: www.example.com
Accept: text/html, application/xhtml+xml, */*
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:123.0) Gecko/20100101 Firefox/123.0
Cookie: sessionid=abcdefg1234567
Connection: keep-alive

```

上述请求头包括了以下字段：

+ **Request-line**：指定使用GET方法请求/index.html资源，并使用HTTP/1.1协议版本。
+ **Host**：指定被请求资源所在主机名或IP地址和端口号。
+ **Accept**：客户端期望接收的媒体类型列表，本例中指定了text/html、application/xhtml+xml和任意类型的文件（_/_）。
+ **User-Agent**：客户端浏览器类型和版本号。
+ **Cookie**：客户端发送给服务器的cookie信息。
+ **Connection**：客户端请求后是否需要保持长连接。

### HTTP 响应头
HTTP响应头包括以下字段：

+ **Status-line**：包含协议版本、状态码和状态消息。
+ **Content-Type**：响应体的MIME类型。
+ **Content-Length**：响应体的字节数。
+ **Set-Cookie**：服务器向客户端发送cookie信息时使用该字段。
+ **Server**：服务器类型和版本号。
+ **Connection**：表示是否需要保持长连接（keep-alive）。

在实际的HTTP报文头中，还可以包含其他可选字段。  
如下是一个http响应头的示例

```plain
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1024
Set-Cookie: sessionid=abcdefg1234567; HttpOnly; Path=/
Server: Apache/2.2.32 (Unix) mod_ssl/2.2.32 OpenSSL/1.0.1e-fips mod_bwlimited/1.4
Connection: keep-alive
```

上述响应头包括了以下字段：

+ **Status-line**：指定HTTP协议版本、状态码和状态消息。
+ **Content-Type**：指定响应体的MIME类型及字符编码格式。
+ **Content-Length**：指定响应体的字节数。
+ **Set-Cookie**：服务器向客户端发送cookie信息时使用该字段。
+ **Server**：服务器类型和版本号。
+ **Connection**：服务器是否需要保持长连接。

## 客户端的编写
客户端每次发送数据都要携带头部信息，所以为了减少每次重新构造头部的开销，我们在客户端的构造函数里将头部信息构造好，作为一个成员放入客户端的类成员里。

```cpp
    client(boost::asio::io_context& io_context,
        const std::string& server, const std::string& path)
        : resolver_(io_context),
        socket_(io_context)
    {
        // Form the request. We specify the "Connection: close" header so that the
        // server will close the socket after transmitting the response. This will
        // allow us to treat all data up until the EOF as the content.
        std::ostream request_stream(&request_);
        request_stream << "GET " << path << " HTTP/1.0\r\n";
        request_stream << "Host: " << server << "\r\n";
        request_stream << "Accept: */*\r\n";
        request_stream << "Connection: close\r\n\r\n";

        size_t pos = server.find(":");
        std::string ip = server.substr(0, pos);
        std::string port = server.substr(pos + 1);

        // Start an asynchronous resolve to translate the server and service names
        // into a list of endpoints.
        resolver_.async_resolve(ip, port,
            boost::bind(&client::handle_resolve, this,
                boost::asio::placeholders::error,
                boost::asio::placeholders::results));
    }
```

我们的客户端构造了一个`request_`成员变量，依次写入请求的路径，主机地址，期望接受的媒体类型，以及每次收到请求后断开连接，也就是短链接的方式。  
接着又异步解析ip和端口，解析成功后调用`handle_resolve`函数。  
`handle_resolve`函数里异步处理连接

```cpp
   void handle_resolve(const boost::system::error_code& err,
        const tcp::resolver::results_type& endpoints)
    {
        if (!err)
        {
            // Attempt a connection to each endpoint in the list until we
            // successfully establish a connection.
            boost::asio::async_connect(socket_, endpoints,
                boost::bind(&client::handle_connect, this,
                    boost::asio::placeholders::error));
        }
        else
        {
            std::cout << "Error: " << err.message() << "\n";
        }
    }
```

处理连接

```cpp
   void handle_connect(const boost::system::error_code& err)
    {
        if (!err)
        {
            // The connection was successful. Send the request.
            boost::asio::async_write(socket_, request_,
                boost::bind(&client::handle_write_request, this,
                    boost::asio::placeholders::error));
        }
        else
        {
            std::cout << "Error: " << err.message() << "\n";
        }
    }
```

在连接成功后，我们首先将头部信息发送给服务器,发送完成后监听对端发送的数据

```cpp
    void handle_write_request(const boost::system::error_code& err)
    {
        if (!err)
        {
            // Read the response status line. The response_ streambuf will
            // automatically grow to accommodate the entire line. The growth may be
            // limited by passing a maximum size to the streambuf constructor.
            boost::asio::async_read_until(socket_, response_, "\r\n",
                boost::bind(&client::handle_read_status_line, this,
                    boost::asio::placeholders::error));
        }
        else
        {
            std::cout << "Error: " << err.message() << "\n";
        }
    }
```

当收到对方数据时，先解析响应的头部信息

```cpp
    void handle_read_status_line(const boost::system::error_code& err)
    {
        if (!err)
        {
            // Check that response is OK.
            std::istream response_stream(&response_);
            std::string http_version;
            response_stream >> http_version;
            unsigned int status_code;
            response_stream >> status_code;
            std::string status_message;
            std::getline(response_stream, status_message);
            if (!response_stream || http_version.substr(0, 5) != "HTTP/")
            {
                std::cout << "Invalid response\n";
                return;
            }
            if (status_code != 200)
            {
                std::cout << "Response returned with status code ";
                std::cout << status_code << "\n";
                return;
            }

            // Read the response headers, which are terminated by a blank line.
            boost::asio::async_read_until(socket_, response_, "\r\n\r\n",
                boost::bind(&client::handle_read_headers, this,
                    boost::asio::placeholders::error));
        }
        else
        {
            std::cout << "Error: " << err << "\n";
        }
    }
```

上面的代码先读出HTTP版本，以及返回的状态码，如果状态码不是200，则返回，是200说明响应成功。接下来把所有的头部信息都读出来。

```cpp
    void handle_read_headers(const boost::system::error_code& err)
    {
        if (!err)
        {
            // Process the response headers.
            std::istream response_stream(&response_);
            std::string header;
            while (std::getline(response_stream, header) && header != "\r")
                std::cout << header << "\n";
            std::cout << "\n";

            // Write whatever content we already have to output.
            if (response_.size() > 0)
                std::cout << &response_;

            // Start reading remaining data until EOF.
            boost::asio::async_read(socket_, response_,
                boost::asio::transfer_at_least(1),
                boost::bind(&client::handle_read_content, this,
                    boost::asio::placeholders::error));
        }
        else
        {
            std::cout << "Error: " << err << "\n";
        }
    }
```

上面的代码逐行读出头部信息，然后读出响应的内容，继续监听读事件读取相应的内容，直到接收到EOF信息，也就是对方关闭，继续监听读事件是因为有可能是长连接的方式，当然如果是短链接，则服务器关闭连接后，客户端也是通过异步函数读取EOF进而结束请求。

```cpp
    void handle_read_content(const boost::system::error_code& err)
    {
        if (!err)
        {
            // Write all of the data that has been read so far.
            std::cout << &response_;

            // Continue reading remaining data until EOF.
            boost::asio::async_read(socket_, response_,
                boost::asio::transfer_at_least(1),
                boost::bind(&client::handle_read_content, this,
                    boost::asio::placeholders::error));
        }
        else if (err != boost::asio::error::eof)
        {
            std::cout << "Error: " << err << "\n";
        }
    }
```

在主函数中调用客户端请求服务器信息, 请求的路由地址为`/`

```cpp
int main(int argc, char* argv[])
{
    try
    {
        boost::asio::io_context io_context;
        client c(io_context, "127.0.0.1:8080", "/");
        io_context.run();
        getchar();
    }
    catch (std::exception& e)
    {
        std::cout << "Exception: " << e.what() << "\n";
    }

    return 0;
}
```

## 服务器设计
为了方便理解，我们从服务器的调用流程讲起

```cpp
int main(int argc, char* argv[])
{
    try
    {
        std::filesystem::path path = std::filesystem::current_path() / "res";

        // 使用 std::cout 输出拼接后的路径
        std::cout << "Path: " << path.string() << '\n';
        std::cout << "Usage: http_server <127.0.0.1> <8080> "<< path.string() <<"\n";
        // Initialise the server.
        http::server::server s("127.0.0.1", "8080", path.string());

        // Run the server until stopped.
        s.run();
    }
    catch (std::exception& e)
    {
        std::cerr << "exception: " << e.what() << "\n";
    }

    return 0;
}
```

主函数里构造了一个server对象，然后调用了run函数使其跑起来。  
run函数其实就是调用了server类成员的ioservice

```cpp
void server::run()
{
    io_service_.run();
}
```

server类的构造函数里初始化一些成员变量，比如acceptor连接器，绑定了终止信号，并且监听对端连接

```cpp
server::server(const std::string& address, const std::string& port,
    const std::string& doc_root)
            : io_service_(),
            signals_(io_service_),
            acceptor_(io_service_),
            connection_manager_(),
            socket_(io_service_),
            request_handler_(doc_root)
    {
            signals_.add(SIGINT);
            signals_.add(SIGTERM);
#if defined(SIGQUIT)
            signals_.add(SIGQUIT);
#endif 
            do_await_stop();
            boost::asio::ip::tcp::resolver resolver(io_service_);
            boost::asio::ip::tcp::endpoint endpoint = *resolver.resolve({ address, port });
            acceptor_.open(endpoint.protocol());
            acceptor_.set_option(boost::asio::ip::tcp::acceptor::reuse_address(true));
            acceptor_.bind(endpoint);
            acceptor_.listen();
            do_accept();
    }
```

接收连接

```cpp
void server::do_accept()
{
    acceptor_.async_accept(socket_,
        [this](boost::system::error_code ec)
        {

            if (!acceptor_.is_open())
            {
                return;
            }

            if (!ec)
            {
                connection_manager_.start(std::make_shared<connection>(
                            std::move(socket_), connection_manager_, request_handler_));
                    }

                do_accept();
            });
        }
```

接收函数里通过`connection_manager_`启动了一个新的连接，用来处理读写函数。  
处理方式和我们之前的写法类似，只是我们之前管理连接用的server，这次用的conneciton_manager

```cpp
void connection_manager::start(connection_ptr c)
{
    connections_.insert(c);
    c->start();
}
```

start函数里处理读写

```cpp
void connection::start()
{
    do_read();
}
```

处理读数据比较复杂，我们分部分解释

```cpp
void connection::do_read()
{
    auto self(shared_from_this());
    socket_.async_read_some(boost::asio::buffer(buffer_),
        [this, self](boost::system::error_code ec, std::size_t bytes_transferred)
    {
        if (!ec)
        {
            request_parser::result_type result;
            std::tie(result, std::ignore) = request_parser_.parse(
                request_, buffer_.data(), buffer_.data() + bytes_transferred);

            if (result == request_parser::good)
            {
                request_handler_.handle_request(request_, reply_);
                do_write();
            }
            else if (result == request_parser::bad)
            {
                reply_ = reply::stock_reply(reply::bad_request);
                do_write();
            }
            else
            {
                do_read();
            }
        }
        else if (ec != boost::asio::error::operation_aborted)
        {
            connection_manager_.stop(shared_from_this());
        }
    });
}
```

 通过`request_parser_`解析请求，然后根据请求结果选择处理请求还是返回错误。

```cpp
std::tuple<result_type, InputIterator> parse(request& req,
               InputIterator begin, InputIterator end)
{
   while (begin != end)
   {
       result_type result = consume(req, *begin++);
       if (result == good || result == bad)
           return std::make_tuple(result, begin);
   }
   return std::make_tuple(indeterminate, begin);
}
```

 parse是解析请求的函数，内部调用了consume不断处理请求头中的数据，其实就是一个逐行解析的过程, consume函数很长，这里就不解释了，其实就是每解析一行就更改一下状态，这样可以继续解析。具体可以看看源码。

 在consume()函数中，根据每个字符输入的不同情况，判断当前所处状态state_，进而执行相应的操作，包括：

+ 将HTTP请求方法、URI和HTTP版本号解析到request结构体中。
+ 解析每个请求头部字段的名称和值，并将其添加到request结构体中的headers vector中。
+ 如果输入字符为`\r\n`，则修改状态以开始下一行的解析。

最后，返回一个枚举类型request_parser::result_type作为解析结果，包括indeterminate、good和bad三种状态。其中，indeterminate表示还需要继续等待更多字符输入；good表示成功解析出了一个完整的HTTP请求头部；bad表示遇到无效字符或格式错误，解析失败。

解析完成头部后会调用处理请求的函数,这里只是简单的写了一个作为资源服务器解析资源请求的逻辑，具体可以看源码。

```cpp
void request_handler::handle_request(const request& req, reply& rep)
{
    // Decode url to path.
    std::string request_path;
    if (!url_decode(req.uri, request_path))
    {
        rep = reply::stock_reply(reply::bad_request);
        return;
    }
    // Request path must be absolute and not contain "..".
    if (request_path.empty() || request_path[0] != '/'
        || request_path.find("..") != std::string::npos)
    {
        rep = reply::stock_reply(reply::bad_request);
        return;
    }

    // If path ends in slash (i.e. is a directory) then add "index.html".
    if (request_path[request_path.size() - 1] == '/')
    {
        request_path += "index.html";
    }

    // Determine the file extension.
    std::size_t last_slash_pos = request_path.find_last_of("/");
    std::size_t last_dot_pos = request_path.find_last_of(".");
    std::string extension;
    if (last_dot_pos != std::string::npos && last_dot_pos > last_slash_pos)
    {
        extension = request_path.substr(last_dot_pos + 1);
    }

    // Open the file to send back.
    std::string full_path = doc_root_ + request_path;
    std::ifstream is(full_path.c_str(), std::ios::in | std::ios::binary);
    if (!is)
    {
        rep = reply::stock_reply(reply::not_found);
        return;
    }

    // Fill out the reply to be sent to the client.
    rep.status = reply::ok;
    char buf[512];
    while (is.read(buf, sizeof(buf)).gcount() > 0)
        rep.content.append(buf, is.gcount());
        rep.headers.resize(2);
        rep.headers[0].name = "Content-Length";
        rep.headers[0].value = std::to_string(rep.content.size());
        rep.headers[1].name = "Content-Type";
        rep.headers[1].value = mime_types::extension_to_type(extension);
    }
```

上述代码根据url中的`.`来做切割，获取请求的文件类型，然后根据`/`切割url，获取资源目录，最后返回资源文件。  
如果你想实现普通的路由请求返回json或者text格式，可以重写处理请求的逻辑。

