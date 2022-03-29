---
title: 初识WSGI接口
tags: python
abbrlink: 4b88f152
date: 2019-09-15 21:41:54
---

---
title: 初识WSGI接口
date: 2019-09-15 21:41:54
tags: python
---
# WSGI
WSGI全称为**Web Server Gateway Interface**，WSGI允许web框架和web服务器分开，可以混合匹配web服务器和web框架，选择一个适合的配对。比如,可以在Gunicorn 或者 Nginx/uWSGI 或者 Waitress上运行 Django, Flask, 或 Pyramid。

web服务器必须具备wsgi接口，所有的现代Python web框架都以具备wsgi接口，它不让你对代码作修改就能使服务器和web框架协同工作。

其他语言也有类似的接口：java中的servlet

# 定义WSGI接口
WSGI接口的定义非常简单，它只是要求web开发者实现一个函数，就可以相应http请求。我们来看一个最简单的Web版本的“Hello, World!”：
```python
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return 'Hello World!'
```
上面的 **application()** 函数就是符合WSGI标准的一个HTTP处理函数，它接收两个参数：
- environ：一个包含所有http请求信息的一个字典对象。
- start_response：一个发送HTTP响应的函数，它接收两个参数：
 	- http响应码。
 	- 一组list表示的HTTP header，每个header用一个包含两个str的tuple表示。

函数的返回值Hello, World!将作为HTTP响应的Body发送给浏览器。
**application()函数必须由WSGI服务器来调用。**

