---
title: python之requests库快速上手
date: 2022-07-05 09:28:04
tags: 爬虫
---

> 之前使用的python自带的urllib和request库相比，简直就是弱爆了，request使用非常简单，功能又强大



# 安装

```shell
pip install requests 
```



# response的属性以及类型

- 类型 ：`requests.models.Response`

- `r.text` : 获取网站源码 
- `r.json()` ：把相应的json字符串转换成python对象
- `r.content` ：响应的字节类型 

- `r.encoding` ：访问或定制编码方式 

- `r.url` ：获取请求的url 

- `r.status_code` ：响应的状态码 

- `r.headers` ：响应的头信息 



# get请求

```python
import requests
url = 'https://httpbin.org/get'
headers = {
    'User-agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.163 Safari/535.1 '
}
data = { 'name':'张三' }
r = requests.get(url,headers=headers,params=data)
print(r.text)
```

- 发送get请求时，参数使用params传递，requests会帮我们拼接在url上 

- 参数无需urlencode编码 ,requests库会自动帮我们完成

- 和urllib相比自定义请求头不需要请求对象的定制 

- 请求资源路径中`?`可加可不加

# post请求

```python
import requests
url = 'https://httpbin.org/post'
headers = {
    'User-agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.163 Safari/535.1 '
}
data = {
    'name':'张三',
    'pwd':'123456'
}
r = requests.post(url,headers=headers,data=data)
print(r.text)
```

**注意：**

- get请求的参数名字是`params`，post请求的参数的名字是`data`

- 与urllib相比，发送post请求时，发送的数据不需要手动编码 

# 代理

```python
import requests
url = 'http://httpbin.org/get'
headers = {
    'User-agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.163 '
                  'Safari/535.1 '
}
data = { 'name':'张三' }
# 设置代理
proxy={
    'http':'112.6.117.178:8085'
}
r = requests.get(url,headers=headers,params=data,proxies=proxy)
print(r.text)
```

# session对象

通过同一个Session对象发送的请求，会自动管理由服务器响应的cookie，使其在同一个session域中

```python
import requests
s = requests.Session()

s.get('https://httpbin.org/cookies/set/sessioncookie/123456789')
r = s.get('https://httpbin.org/cookies')

print(r.text)
# '{"cookies": {"sessioncookie": "123456789"}}'
```

