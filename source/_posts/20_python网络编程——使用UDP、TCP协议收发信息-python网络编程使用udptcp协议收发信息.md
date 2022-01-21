---
title: python网络编程——使用UDP、TCP协议收发信息
date: 2019-08-25 15:24:00
tags: python
---



# UDP

UDP是面向无连接的通讯协议，UDP数据包括目的端口号和源端口号信息，由于通讯不需要连接，所以可以实现广播发送。 UDP传输数据时有大小限制，每个被传输的数据报必须限定在64KB之内。 UDP是一个不可靠的协议，发送方所发送的数据报并不一定以相同的次序到达接收方。

udp通信模型中，在通信开始之前，不需要建立相关的链接，只需要发送数据即可，类似于生活中，"写信"。
**客户端：**
```python
from socket import socket,AF_INET,SOCK_DGRAM
# 创建套接字，SOCK_DGRAM使用udp协议
udp = socket(AF_INET, SOCK_DGRAM)
# 目的端口和ip
ip = "127.0.0.1"
port = 8080
# 循环从键盘输入发送消息
while True:
    data = input("请输入发送的数据：")
    udp.sendto(data.encode("utf-8"), (ip, port))
```
**服务端：**
```python
from socket import socket, AF_INET, SOCK_DGRAM

udp = socket(AF_INET, SOCK_DGRAM)
# 绑定端口，服务端必须要绑定端口
udp.bind(("", 8080))

while True:
    # 接受数据，每次接受1024字节
    recvData = udp.recvfrom(1024)
    # 拆包
    data, info = recvData
    # 打印
    print("[%s]:%s" % (info, data.decode("utf-8")))
```
# TCP
udp通信模型中，在通信开始之前，一定要先建立相关的链接，才能发送数据，类似于生活中，"打电话"。
**客户端：**
```python
from socket import socket,AF_INET,SOCK_STREAM
# 创建套接字，SOCK_STREAM表示使用tcp协议
clientSocket = socket(AF_INET,SOCK_STREAM)
# 连接服务器
clientSocket.connect(("127.0.0.1",8080))
# 发送数据
while True:
    s = input("请输入要发送的数据：")
    clientSocket.send(s.encode("utf-8"))
```
**服务端：**
```python
from socket import socket, AF_INET, SOCK_STREAM

tcp = socket(AF_INET, SOCK_STREAM)
# 绑定端口
tcp.bind(("", 8080))
# listen的参数代表可建立socket连接的最大个数  windows，mac 此连接参数有效  Linux 此连接参数无效，默认最大
tcp.listen()

# 有新的客户端连接时，
# clientSocket表示一个新的套接字
# clientInfo 表示新客户端的ip及端口号
while True:
    clientSocket, clientInfo = tcp.accept()
    try:
        while True:
            recvData = clientSocket.recv(1024)
            # 如果接受的的数据为空就退出
            if not recvData:
                break
            print("%s:%s" % (str(clientInfo), recvData.decode("utf-8")))
    finally:
        clientSocket.close()
```
