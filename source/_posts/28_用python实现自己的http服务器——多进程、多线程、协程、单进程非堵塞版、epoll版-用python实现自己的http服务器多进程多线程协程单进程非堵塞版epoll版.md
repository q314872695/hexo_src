---
title: 用python实现自己的http服务器——多进程、多线程、协程、单进程非堵塞版、epoll版
tags: python
abbrlink: aba931f3
date: 2019-08-28 15:24:00
---



# 了解http协议

## http请求头
```python
GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Sec-Fetch-Site: none
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
```
**最主要的头两行分析如下：**
- GET表示一个读取请求，将从服务器获得网页数据，/表示URL的路径，URL总是以/开头，/就表示首页，最后的HTTP/1.1指示采用的HTTP协议版本是1.1。
- 目前HTTP协议的版本就是1.1，但是大部分服务器也支持1.0版本，主要区别在于1.1版本允许多个HTTP请求复用一个TCP连接，以加快传输速度。
- Host: `www.baidu.com` 表示请求的域名是 `www.baidu.com` 。如果一台服务器有多个网站，服务器就需要通过Host来区分浏览器请求的是哪个网站。

## http响应头
```python
HTTP/1.1 200 OK
Bdpagetype: 2
Bdqid: 0x8ef7ae5901149cf7
Cache-Control: private
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html;charset=utf-8
Date: Wed, 28 Aug 2019 01:59:49 GMT
Expires: Wed, 28 Aug 2019 01:59:48 GMT
Server: BWS/1.1
Set-Cookie: BDSVRTM=249; path=/
Set-Cookie: BD_HOME=1; path=/
Set-Cookie: H_PS_PSSID=1426_21111_20697_29522_29518_29099_29568_29220_26350; path=/; domain=.baidu.com
Strict-Transport-Security: max-age=172800
X-Ua-Compatible: IE=Edge,chrome=1
Transfer-Encoding: chunked
```
**说明：**
- 200表示一个成功的响应，后面的OK是说明。
- Content-Type指示响应的内容，这里是text/html表示HTML网页。
- 请求头和响应头通过\r\n来换行。
- 响应头和body响应体中也通过\r\n来分隔。

# 简单的http服务器
有多简单呢？运行程序后打开浏览器，只能显示hello world。
```python
import socket


def service_client(new_socket):
    # 接受浏览器发过来的http请求
    # GET / HTTP/1.1
    request = new_socket.recv(1024)
    print(request)
    # 返回http响应
    resposne = "HTTP/1.1 200 OK\r\n"
    resposne += "\r\n"
    resposne += "<h1>hello world</h1>"
    new_socket.send(resposne.encode("utf-8"))

    # 关闭套接字
    new_socket.close()


def main():
    # 创建套接字
    http_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 防止端口被占用无法启动程序
    http_server.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)
    # 绑定端口
    http_server.bind(("", 80))
    # 变为监听套接字
    http_server.listen(128)
    while True:
        # 等在新客户端连接
        client, info = http_server.accept()
        # 为这个客户端服务
        service_client(client)


if __name__ == "__main__":
    main()

```
# 单进程http服务器
它上之前有一个升级就是可以返回静态的html页面。
```python
import socket
import re


def service_client(new_socket):
    # 接受浏览器发过来的http请求
    # GET / HTTP/1.1
    request = new_socket.recv(1024).decode("utf-8")
    # print(request)
    request_lines = request.splitlines()
    req = re.match(r"[^/]+(/\S*)", request_lines[0])
    file_name: str = ""
    if req:
        file_name = req.group(1)
        if file_name == "/":
            file_name = "/index.html"
        print(file_name)
    # print(request_lines)
    # 返回http响应

    try:
    	# 打开要请求的html文件，并返回给客户端。网页在当前路径的html文件夹里面。
        with open("./html" + file_name, "r", encoding="utf-8") as f:
            resposne = "HTTP/1.1 200 OK\r\n"
            resposne += "\r\n"
            # resposne += "<h1>hello world</h1>"
            resposne += f.read()
    except Exception as e:
        resposne = "HTTP/1.1 400 NOT FOUND\r\n"
        resposne += "\r\n"
        resposne += "--file not found--"

    new_socket.send(resposne.encode("utf-8"))
    # 关闭套接字
    new_socket.close()


def main():
    # 创建套接字
    http_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 防止端口被占用无法启动程序
    http_server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定端口
    http_server.bind(("", 80))
    # 变为监听套接字
    http_server.listen(128)
    while True:
        # 等在新客户端连接
        client, info = http_server.accept()
        # 为这个客户端服务
        service_client(client)


if __name__ == "__main__":
    main()
```
# 多进程服务器
http服务器是完成了，但是如果同时有好多人访问的话它反应就会非常慢，所有又在之前的基础上做了升级，增加服务器的并发能力。
```python
import socket
import re
from multiprocessing import Process

def service_client(new_socket):
    # 接受浏览器发过来的http请求
    # GET / HTTP/1.1
    request = new_socket.recv(1024).decode("utf-8")
    # print(request)
    request_lines = request.splitlines()
    req = re.match(r"[^/]+(/\S*)", request_lines[0])
    file_name: str = ""
    if req:
        file_name = req.group(1)
        if file_name == "/":
            file_name = "/index.html"
        print(file_name)
    # print(request_lines)
    # 返回http响应

    try:
        with open("./html" + file_name, "r", encoding="utf-8") as f:
            resposne = "HTTP/1.1 200 OK\r\n"
            resposne += "\r\n"
            # resposne += "<h1>hello world</h1>"
            resposne += f.read()
    except Exception as e:
        resposne = "HTTP/1.1 400 NOT FOUND\r\n"
        resposne += "\r\n"
        resposne += "--file not found--"

    new_socket.send(resposne.encode("utf-8"))
    # new_socket.send(body)
    # 关闭套接字
    new_socket.close()


def main():
    # 创建套接字
    http_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 防止端口被占用无法启动程序
    http_server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定端口
    http_server.bind(("", 80))
    # 变为监听套接字
    http_server.listen(128)
    while True:
        # 等在新客户端连接
        client, info = http_server.accept()
        # 开启一个子进程为这个客户端服务
        p = Process(target=service_client,args=(client,))
        p.start()
        # 子进程会复制主进程发的资源，故把主进程的socket关闭。
        client.close()

if __name__ == "__main__":
    main()
```
# 多线程服务器
我们知道进程耗费资源是非常大的，所以这次使用了耗费资源小的线程来实现多任务。
```python
import socket
import re
from threading import Thread


def service_client(new_socket):
    # 接受浏览器发过来的http请求
    # GET / HTTP/1.1
    request = new_socket.recv(1024).decode("utf-8")
    # print(request)
    request_lines = request.splitlines()
    req = re.match(r"[^/]+(/\S*)", request_lines[0])
    file_name: str = ""
    if req:
        file_name = req.group(1)
        if file_name == "/":
            file_name = "/index.html"
        print(file_name)
    # print(request_lines)
    # 返回http响应

    try:
        with open("./html" + file_name, "r", encoding="utf-8") as f:
            resposne = "HTTP/1.1 200 OK\r\n"
            resposne += "\r\n"
            # resposne += "<h1>hello world</h1>"
            resposne += f.read()
    except Exception as e:
        resposne = "HTTP/1.1 400 NOT FOUND\r\n"
        resposne += "\r\n"
        resposne += "--file not found--"

    new_socket.send(resposne.encode("utf-8"))
    # new_socket.send(body)
    # 关闭套接字
    new_socket.close()


def main():
    # 创建套接字
    http_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 防止端口被占用无法启动程序
    http_server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定端口
    http_server.bind(("", 80))
    # 变为监听套接字
    http_server.listen(128)
    while True:
        # 等在新客户端连接
        client, info = http_server.accept()
        # 开启一个子线程为这个客户端服务
        p = Thread(target=service_client, args=(client,))
        p.start()


if __name__ == "__main__":
    main()
```
# gevent协程版的服务器
协程在一个线程中执行，减少了线程之间的切换，多线程的升级版，拥有更好的处理能力。
```python
import socket
import re
import gevent
from gevent import monkey

monkey.patch_all()

def service_client(new_socket):
    # 接受浏览器发过来的http请求
    # GET / HTTP/1.1
    request = new_socket.recv(1024).decode("utf-8")
    # print(request)
    request_lines = request.splitlines()
    req = re.match(r"[^/]+(/\S*)", request_lines[0])
    file_name: str = ""
    if req:
        file_name = req.group(1)
        if file_name == "/":
            file_name = "/index.html"
        print(file_name)
    # print(request_lines)
    # 返回http响应

    try:
        with open("./html" + file_name, "r", encoding="utf-8") as f:
            resposne = "HTTP/1.1 200 OK\r\n"
            resposne += "\r\n"
            # resposne += "<h1>hello world</h1>"
            resposne += f.read()
    except Exception as e:
        resposne = "HTTP/1.1 400 NOT FOUND\r\n"
        resposne += "\r\n"
        resposne += "--file not found--"

    new_socket.send(resposne.encode("utf-8"))
    # new_socket.send(body)
    # 关闭套接字
    new_socket.close()


def main():
    # 创建套接字
    http_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 防止端口被占用无法启动程序
    http_server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定端口
    http_server.bind(("", 80))
    # 变为监听套接字
    http_server.listen(128)
    while True:
        # 等在新客户端连接
        client, info = http_server.accept()
        # 为这个客户端服务
        gevent.spawn(service_client, client)


if __name__ == "__main__":
    main()
```
# 单进程非堵塞版长连接的服务器
从这个例子中来引入epoll版，它的性能应该要比协程的好，与之前所有服务器的不同之处就是采用了长连接，通过响应头中的Content-Length来指定响应体的长度，从而让浏览器知道页面数据传输完成以后自动在同一个套接字连接中继续发送其他资源文件的请求，效率较高。之前的都是短连接，只要传输完当前文件就关闭这个套接字。
```python
import socket
import re


def service_client(new_socket: object, request: str):
    # 接受浏览器发过来的http请求
    # GET / HTTP/1.1
    # request = new_socket.recv(1024).decode("utf-8")
    # print(request)
    request_lines = request.splitlines()
    req = re.match(r"[^/]+(/\S*)", request_lines[0])
    file_name: str = ""
    if req:
        file_name = req.group(1)
        if file_name == "/":
            file_name = "/index.html"
        print(file_name)
    # print(request_lines)
    # 返回http响应

    try:
        with open("./html" + file_name, "r", encoding="utf-8") as f:
            resposne_body: str = f.read()
            resposne_header: str = "HTTP/1.1 200 OK\r\n"
            resposne_header += "Content-Length:%d\r\n" % len(resposne_body)
            resposne_header += "\r\n"
            # resposne += "<h1>hello world</h1>"
            resposne = resposne_header + resposne_body
    except Exception as e:
        resposne = "HTTP/1.1 400 NOT FOUND\r\n"
        resposne += "\r\n"
        resposne += "--file not found--"

    new_socket.send(resposne.encode("utf-8"))
    # new_socket.send(body)
    # 关闭套接字
    # new_socket.close()


def main():
    # 创建套接字
    http_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 防止端口被占用无法启动程序
    http_server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定端口
    http_server.bind(("", 80))
    # 变为监听套接字
    http_server.listen(128)
    # 设置套接字为非堵塞方式
    http_server.setblocking(False)
    socket_list: list = []
    while True:
        try:
            # 等在新客户端连接
            client, info = http_server.accept()
            # 为这个客户端服务
            # gevent.spawn(service_client, client)
        except Exception as e:
            # print(e)
            pass
        else:
            client.setblocking(False)
            socket_list.append(client)

        for socket_client in socket_list:
            try:
                recv_data: str = socket_client.recv(1024).decode("utf-8")
            except Exception as e:
                # print(e)
                pass
            else:
                if recv_data:
                    service_client(socket_client, recv_data)
                else:
                    socket_list.remove(socket_client)
                    socket_client.close()


if __name__ == "__main__":
    main()

```
# eopll版的服务器
这个版本是性能最高的服务器。它的基本原理就是select，poll，epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。
```python
import socket
import re
import select


def service_client(new_socket: object, request: str):
    # 接受浏览器发过来的http请求
    # GET / HTTP/1.1
    # request = new_socket.recv(1024).decode("utf-8")
    # print(request)
    request_lines = request.splitlines()
    req = re.match(r"[^/]+(/\S*)", request_lines[0])
    file_name: str = ""
    if req:
        file_name = req.group(1)
        if file_name == "/":
            file_name = "/index.html"
        print(file_name)
    # print(request_lines)
    # 返回http响应

    try:
        with open("./html" + file_name, "r", encoding="utf-8") as f:
            resposne_body: str = f.read()
            resposne_header: str = "HTTP/1.1 200 OK\r\n"
            resposne_header += "Content-Length:%d\r\n" % len(resposne_body)
            resposne_header += "\r\n"
            # resposne += "<h1>hello world</h1>"
            resposne = resposne_header + resposne_body
    except Exception as e:
        resposne = "HTTP/1.1 400 NOT FOUND\r\n"
        resposne += "\r\n"
        resposne += "--file not found--"

    new_socket.send(resposne.encode("utf-8"))
    # new_socket.send(body)
    # 关闭套接字
    # new_socket.close()


def main():
    # 创建套接字
    http_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 防止端口被占用无法启动程序
    http_server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    # 绑定端口
    http_server.bind(("", 80))
    # 变为监听套接字
    http_server.listen(128)
    # 设置套接字为非堵塞方式
    http_server.setblocking(False)
    # 创建一个epoll对象
    epl = select.epoll()
    # 将监听套接字对应的fd(文件描述符)注册到epoll中
    epl.register(http_server.fileno(), select.EPOLLIN)
    # 存储fd文件描述符和套接字的对应关系
    fd_event_dict: dict = {}
    while True:
        # 默认会堵塞，知道os检测到数据到来，通过事件通知方式告诉这个程序，此时才会解堵塞
        fd_event_list: list = epl.poll()  # [(套接字对应的文件描述符，这个文件描述符是什么事件),...]
        for fd, event in fd_event_list:
            # 如果是监听套接字有数据过来，即等待新的客户端连接
            if fd == http_server.fileno():
                client, info = http_server.accept()
                # 将新的套接字注册到epoll中
                epl.register(client.fileno(), select.EPOLLIN)
                # 把文件描述符和套接字的对应关系存入字典
                fd_event_dict[client.fileno()] = client
            elif event == select.EPOLLIN:
                # 判断已连接的套接字是否有数据发过来
                recv_data: str = fd_event_dict[fd].recv(1024).decode("utf-8")
                if recv_data:
                    service_client(fd_event_dict[fd], recv_data)
                else:
                    fd_event_dict[fd].close()
                    epl.unregister(fd)
                    del fd_event_dict[fd]


if __name__ == '__main__':
    main()

```
**I/O 多路复用的特点：**
通过一种机制使一个进程能同时等待多个文件描述符，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，epoll()函数就可以返回。 所以, IO多路复用，本质上不会有并发的功能，因为任何时候还是只有一个进程或线程进行工作，它之所以能提高效率是因为select\epoll 把进来的socket放到他们的 '监视' 列表里面，当任何socket有可读可写数据立马处理，那如果select\epoll 手里同时检测着很多socket， 一有动静马上返回给进程处理，总比一个一个socket过来,阻塞等待,处理高效率。
