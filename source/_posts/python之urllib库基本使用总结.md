---
title: python之urllib库基本使用总结
tags: 爬虫
abbrlink: 95a5f622
date: 2022-07-03 08:57:27
---



# urllib模块介绍

在 Python3 中 urllib 模块包括如下内容。

- `urllib.request`：请求模块，用于打开和读取 URL；
- `urllib.error`：异常处理模块，捕获 `urllib.error` 抛出异常；
- `urllib.parse`：URL 解析，爬虫程序中用于处理 URL 地址；
- `urllib.robotparser`：解析 robots.txt 文件，判断目标站点哪些内容可爬，哪些不可以爬，但是用的很少。



# 快速上手

访问百度，并打印源码

```python
from urllib.request import urlopen

url='http://www.baidu.com'
response = urlopen(url)
content = response.read() # 默认返回的二进制类型
print(content)

# 打印结果如下（内容太多，显示部分）：
b'<!DOCTYPE html><!--STATUS OK--><html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"><meta content="always" name="referrer"><meta name="theme-color" content="#ffffff"><meta name="description" content="\xe5\x85\xa8\xe7\x90\x83\xe9\xa2\x86\xe5\x85\x88\xe7\x9a\x84\xe4\xb8\xad\xe6\x96\x87\xe6\x90\x9c\xe7\xb4\xa2\xe5\xbc\x95\xe6\x93\x8e\xe3\x80\x81\xe8\x87\xb4\xe5\x8a\x9b\xe4\xba\x8e\xe8\xae\xa9\xe7\xbd\x91\xe6\xb0\x91\xe6\x9b\xb4\xe4\xbe\xbf\xe6\x8d\xb7\xe5\x9c\xb0\xe8\x8e\xb7\xe5\x8f\x96\xe4\xbf\xa1\xe6\x81\xaf\xef\xbc\x8c\xe6\x89\xbe\xe5\x88\xb0\xe6\x89\x80\xe6\xb1\x82\xe3\x80\x82\xe7\x99\xbe\xe5\xba\xa6\xe8\xb6\x85\xe8\xbf\x87\xe5\x8d\x83\xe4\xba\xbf\xe7\x9a\x84\xe4\xb8\xad\xe6\x96\x87\xe7\xbd\x91\xe9\xa1\xb5\xe6\x95\xb0\xe6\x8d\xae\xe5\xba\x93\xef\xbc\x8c\xe5\x8f\xaf\xe4\xbb\xa5\xe7\x9e\xac\xe9\x97\xb4\xe6\x89\xbe\xe5\x88\xb0\xe7\x9b\xb8\xe5\x85\xb3\xe7\x9a\x84\xe6\x90\x9c\xe7\xb4\xa2\xe7\xbb\x93\xe6\x9e\x9c\xe3\x80\x82"><link rel="shortcut icon" href="/favicon
```

使用 `urlopen()` 可以得到一个 `HTTPResposne` 类型的对象，它包含有如下常用方法：

- `read()`：以字节形式读取
- `readline()`：读取一行
- `readlines()`：一行一行读取 直至结束
- `status`：获取状态码
- `headers`：获取请求头

# 发送get请求

上面的例子就是发送的get请求，但是你想要携带中文参数就会报错，类似于这样：

```python
from urllib.request import urlopen

url='http://www.baidu.com/s?wd=周杰伦'
response = urlopen(url)
content = response.read()
print(content)
```

错误信息：

`'ascii' codec can't encode characters in position 10-12: ordinal not in range(128)`

url中携带中文必须进行url编码后才行，具体操作如下：

```python
from urllib.parse import quote,urlencode

# 方式一，适用于单个参数
url='http://www.baidu.com/s?wd='+quote('周杰伦')
response = urlopen(url)
content = response.read()
print(content)

# 方式二，适用于多个参数
params={
    'wd':'周杰伦'
}
url='http://www.baidu.com/s?'+urlencode(params) 
response = urlopen(url)
content = response.read()
print(content)
```

# 发送post请求

假设发送一个登录请求，操作如下：

```python
from urllib.request import Request,urlopen
from urllib.parse import urlencode
url='http://localhost:8080/login'
data={
    'username':'admin',
    'password':'123456'
}
# post请求参数必须进行url编码然后转换成字节类型，因为data接受的是一个字节类型的对象
response=urlopen(request,data=urlencode(data).encode('utf-8'))
result=response.read().decode('utf-8')
print(result)
```



**发送get请求和post请求有何区别？**

1. get请求方式的参数必须编码，参数是拼接到url后面，编码之后不需要调用encode方法 
2. post请求方式的参数必须编码，参数是放在请求对象定制的方法中，编码之后需要调用encode方法 

# 自定义请求头

在某些情况下，我们写的代码无法访问指定网站，可能被反爬了，这时，我们就需要自定义请求头修改其中的一个内容，例如`User-Agent`、`Cookie`等等。

```python
from urllib.request import Request,urlopen
# Request类可以扩展更多的请求配置，用于设置请求头
header={
    'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.163 Safari/535.1'
}
url="http://www.baidu.com"
request=Request(url=url, headers=header)
response=urlopen(request)
print(response.read())
```



# ip代理

有时候爬虫爬多了不得不配置代理来使用，否则很容易被封ip，

```python
from urllib.request import Request,urlopen
# Request类可以扩展更多的请求配置，用于设置请求头
header={
    'User-agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.163 Safari/535.1'
}
url="http://4.ipw.cn/" # 测试访问的ip
request=Request(url=url, headers=header)
request.set_proxy("58.20.235.180:9091","http") # 配置代理Ip
print(urlopen(request).read()) # 返回结果与代理ip一致则生效
```

# cookie自动管理

`urllib`默认使用的opener是不支持携带cookie的，需要手动添加`HTTPCookieProcessor`

这样情况适用于在用一个session中发起多个不同的请求，如果不加这三行则认为每个请求都是不同的session

```python
from urllib.request import HTTPCookieProcessor, build_opener, install_opener

# 只要在发请求代码的前面添加上这三行，urllib就会帮我们自动管理cookie啦
processor = HTTPCookieProcessor()
opener = build_opener(processor)
install_opener(opener)
```



