---
title: tomcat8以及8之前中文乱码问题解决
tags:
  - tomcat
abbrlink: 2a0eb34c
date: 2020-05-27 22:59:28
---


**get方式参数乱码**

由于tomcat默认使用`ios-8859-1`编码，而本地编辑器或操作系统使用utf-8编码导致乱码，处理如下：（在tomcat8之后不需要了，默认使用utf-8）
```java
// 先获取乱码的参数
String rname = request.getParameter("rname");
// rname 乱码字符串先按ios-8859-1解码为字节数组，然后在按utf-8进行编码
rname = new String(rname.getBytes(StandardCharsets.ISO_8859_1), StandardCharsets.UTF_8);
```

**post 方式参数乱码**

```java
request.setCharacterEncoding("utf-8"); //按照浏览器的编码方式进行设置
```
**响应消息乱码**

一般是没有告诉浏览器使用哪种编码方式，它就会采用操作系统默认的编码方式，所以就导致乱码
```java
// 告诉浏览器以utf-8方式进行编码
response.setContentType("text/html;charset=utf-8");
```

