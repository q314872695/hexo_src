---
title: java中inputStream read()阻塞解决
date: 2020-05-14 22:43:16
tags: java
---

在之前的案例中，程序执行到服务端中`InputStream`中的`read()`就阻塞了，其原因是`read()`能够正常读取文件流，因为文件总是有大小的，所以总能读到末尾返回-1，但是在网络通讯中，完全无法判断在哪结束，也无法抛出异常，所以，并不会返回任何数值。，网上搜了半天，现有如下解决方案：
1. 第一种就是在客户端`write()`完数据后，需要调用`Socket`类中的`public void shutdownOutput()`方法，其功能是禁用此套接字的输出流。对于 TCP 套接字，任何以前写入的数据都将被发送，并且后跟 TCP 的正常连接终止序列。简单来说就是再给服务端发送一个结束的标记，告诉服务端我已经发送完了，服务端中`read()`方法的阻塞就会被解除。例如：
```java
// 只粘贴部分关键代码
int len=0;
byte[] bytes = new byte[1024];
while ((len=fis.read(bytes))!=-1){
	os.write(bytes, 0, len);
}
socket.shutdownOutput();
```
2. 第二种就是修改服务端中的代码，客服端和平常一样（就是把`socket.shutdownOutput()`删掉），在服务端读取数据的while循环中加一个判断，当读取的字节数小于1024时，结束循环，说明已经读取结束。例如：
```java
 int len=0;
byte[] bytes = new byte[1024];
while ((len = is.read(bytes)) != -1) {
	fos.write(bytes, 0, len);
	if (len < 1024) {
 		break;
	}
}
```

