---
title: Servlet中的Request和Response对象
date: 2020-05-21 20:30:40
tags:
- java
- servlet
---

> request和response对象是由服务器创建的，request对象是来获取请求消息，response对象是来设置响应消息
# Request对象
## HTTP请求
### 概念：Hyper Text Transfer Protocol 超文本传输协议
* 传输协议：定义了，客户端和服务器端通信时，发送数据的格式
* 特点：
	1. 基于TCP/IP的高级协议
	2. 默认端口号:80
	3. 基于请求/响应模型的:一次请求对应一次响应
	4. 无状态的：每次请求之间相互独立，不能交互数据

* 历史版本：
	* 1.0：每一次请求响应都会建立新的连接
	* 1.1：复用连接

### 请求消息数据格式
1. 请求行`GET /login.html HTTP/1.1`
	* 请求方式：
		* HTTP协议有7中请求方式，常用的有2种
			* GET：
				1. 请求参数在请求行中，在url后。
				2. 请求的url长度有限制的
				3. 不太安全
			* POST：
				1. 请求参数在请求体中
				2. 请求的url长度没有限制的
				3. 相对安全
2. 请求头：客户端浏览器告诉服务器一些信息
	* 常见的请求头：
		1. `User-Agent`：浏览器告诉服务器，我访问你使用的浏览器版本信息
			* 可以在服务器端获取该头的信息，解决浏览器的兼容性问题
		2. Referer：`http://localhost/login.html`
			* 告诉服务器，我(当前请求)从哪里来？
				* 作用：
					1. 防盗链：
					2. 统计工作：
3. 请求空行：就是用于分割POST请求的请求头，和请求体的。
4. 请求体(正文)：
	* 封装POST请求消息的请求参数的
### 字符串格式：
```
POST /login.html	HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: http://localhost/login.html
Connection: keep-alive
Upgrade-Insecure-Requests: 1
		
username=zhangsan
```
## Request

### request对象继承体系结构：	
```
ServletRequest		-- 接口
	|	继承
HttpServletRequest	-- 接口
	|	实现
org.apache.catalina.connector.RequestFacade 类(由tomcat实现)
```

### request功能：
#### 获取请求消息数据
1. 获取请求行数据
	* `GET /day14/demo1?name=zhangsan HTTP/1.1`
	* 方法：
		1. 获取请求方式 ：`GET`
			* `String getMethod()`  
		2. **获取虚拟目录**：/day14
			* `String getContextPath()`
		3. 获取Servlet路径: /demo1
			* `String getServletPath()`
		4. 获取get方式请求参数：name=zhangsan
			* `String getQueryString()`
		5. **获取请求URI**：/day14/demo1
			* `String getRequestURI()`：`/day14/demo1`统一资源标识符
			* `StringBuffer getRequestURL()`：`http://localhost/day14/demo1`统一资源定位符
		6. 获取协议及版本：HTTP/1.1
			* `String getProtocol()`

		7. 获取客户机的IP地址：
			* `String getRemoteAddr()`
				
2. 获取请求头数据
	* 方法：
		* `String getHeader(String name)`:通过请求头的名称获取请求头的值
		* `Enumeration<String> getHeaderNames()`:获取所有的请求头名称
		
3. 获取请求体数据:
	* 请求体：只有`POST`请求方式，才有请求体，在请求体中封装了`POST`请求的请求参数
	* 步骤：
		1. 获取流对象
			*  `BufferedReader getReader()`：获取字符输入流，只能操作字符数据
			*  `ServletInputStream getInputStream()`：获取字节输入流，可以操作所有类型数据
				* 在文件上传知识点后讲解
			2. 再从流对象中拿数据
#### 其他功能：
1. **获取请求参数通用方式**：不论get还是post请求方式都可以使用下列方法来获取请求参数
	1. `String getParameter(String name)`:根据参数名称获取参数值    username=zs&password=123
	2. `String[] getParameterValues(String name)`:根据参数名称获取参数值的数组  hobby=xx&hobby=game		
	3. `Enumeration<String> getParameterNames()`:获取所有请求的参数名称
	4. `Map<String,String[]> getParameterMap()`:获取所有参数的map集合

> 中文乱码问题：
get方式：tomcat 8 已经将get方式乱码问题解决了（url会默认使用utf-8）
post方式：会乱码，解决：在获取参数前，设置request的编码request.setCharacterEncoding("utf-8");

2. **请求转发**：一种在服务器内部的资源跳转方式
	1. 步骤：
		1. 通过request对象获取请求转发器对象：`RequestDispatcher getRequestDispatcher(String path)`
		2. 使用RequestDispatcher对象来进行转发：`forward(ServletRequest request, ServletResponse response) `
	2. 特点：
		1. 浏览器地址栏路径不发生变化
		2. 只能转发到当前服务器内部资源中。
		3. 转发是一次请求
3. **共享数据**：
	* 域对象：一个有作用范围的对象，可以在范围内共享数据
	* request域：代表一次请求的范围，一般用于请求转发的多个资源中共享数据
	* 方法：
		1. `void setAttribute(String name,Object obj)`:存储数据
		2. `Object getAttitude(String name)`:通过键获取值
		3. `void removeAttribute(String name)`:通过键移除键值对
		4. 获取`ServletContext`：
			* `ServletContext getServletContext()`

# Response对象
## HTTP响应头
### 数据格式
1. 响应行
	1. 组成：协议/版本 响应状态码 状态码描述
	2. 响应状态码：服务器告诉客户端浏览器本次请求和响应的一个状态。
		1. 状态码都是3位数字 
		2. 分类：
			1. 1xx：服务器就收客户端消息，但没有接受完成，等待一段时间后，发送1xx多状态码
			2. 2xx：成功。代表：200
			3. 3xx：重定向。代表：302(重定向)，304(访问缓存)
			4. 4xx：客户端错误。
				* 代表：
					* 404（请求路径没有对应的资源） 
					* 405：请求方式没有对应的doXxx方法
			5. 5xx：服务器端错误。代表：500(服务器内部出现异常)
2. 响应头：
	1. 格式：头名称： 值
	2. 常见的响应头：
		1. `Content-Type`：服务器告诉客户端本次响应体数据格式以及编码格式
		2. `Content-disposition`：服务器告诉客户端以什么格式打开响应体数据
			* `in-line`:默认值,在当前页面内打开
			* `attachment;filename=xxx`：以附件形式打开响应体。文件下载
3. 响应空行
4. 响应体：传输的数据
5. 响应字符串格式
```
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 101
Date: Wed, 06 Jun 2018 07:08:42 GMT

<html>
<head>
<title>$Title$</title>
</head>
<body>
hello , response
</body>
</html>
```
## Response
### 功能：设置响应消息
1. 设置响应行
	1. 格式：`HTTP/1.1 200 OK`
	2. 设置状态码：`setStatus(int sc)` 
2. 设置响应头：`setHeader(String name, String value)` 
		
3. 设置响应体：
	* 使用步骤：
		1. 获取输出流
			* 字符输出流：`PrintWriter getWriter()`

			* 字节输出流：`ServletOutputStream getOutputStream()`
		2. 使用输出流，将数据输出到客户端浏览器
### 案例：
#### 重定向
1. 代码实现
```java
response.sendRedirect("/day15/responseDemo2");
```
2. 重定向的特点:redirect
	1. 地址栏发生变化
	2. 重定向可以访问其他站点(服务器)的资源
	3. 重定向是两次请求。不能使用request对象来共享数据
3. 转发的特点：forward
	1. 转发地址栏路径不变
	2. 转发只能访问当前服务器下的资源
	3. 转发是一次请求，可以使用request对象来共享数据
#### 服务器输出字符数据到浏览器
* 步骤：
	1. 获取字符输出流
	2. 输出数据

* 注意：
	* 乱码问题：
		1. `PrintWriter pw = response.getWriter();`获取的流的默认编码是ISO-8859-1
		2. 设置该流的默认编码
		3. 告诉浏览器响应体使用的编码
```java
//设置编码，是在获取流之前设置
response.setContentType("text/html;charset=utf-8");
```


