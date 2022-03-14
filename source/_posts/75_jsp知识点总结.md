---
title: jsp知识点总结
date: 2020-05-23 17:09:38
tags:
- java
- jsp
---

# JSP
## 概念
Java Server Pages： java服务器端页面
## 定义java代码的方式
1. `<%  代码 %>`：定义的java代码，在service方法中。service方法中可以定义什么，该脚本中就可以定义什么。
2. `<%! 代码 %>`：定义的java代码，在jsp转换后的java类的成员位置。
3. `<%= 代码 %>`：定义的java代码，会输出到页面上。输出语句中可以定义什么，该脚本中就可以定义什么。

## 指令
- 作用：用于配置JSP页面，导入资源文件
- 格式：
`<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ... %>`
* 分类：
	1. `page`： 配置JSP页面的
		* `contentType`：等同于`response.setContentType()`
			1. 设置响应体的mime类型以及字符集
			2. 设置当前jsp页面的编码（只能是高级的IDE才能生效，如果使用低级工具，则需要设置`pageEncoding`属性设置当前页面的字符集）
		* `import`：导包
			* `errorPage`：当前页面发生异常后，会自动跳转到指定的错误页面
			* `isErrorPage`：标识当前也是是否是错误页面。
				* `true`：是，可以使用内置对象`exception`
				* `false`：否。默认值。不可以使用内置对象`exception`
	2. `include`： 页面包含的。导入页面的资源文件
		* `<%@include file="top.jsp"%>`
	3. `taglib`： 导入资源，例如导入JSTL标签库
		* `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`
		* prefix：前缀，自定义的
## 注释:
1. html注释：`<!-- -->`只能注释html代码片段
2. jsp注释：推荐使用`<%-- --%>`可以注释所有
## 内置对象
在jsp页面中不需要创建，直接使用的对象

|变量名|真实类型|作用|
|-------|-------|-------|
|pageContext|PageContext|当前页面共享数据，还可以获取其他八个内置对象|
|request|HttpServletRequest|一次请求访问的多个资源(转发)|
|session|HttpSession|一次会话的多个请求间|
|application|ServletContext|所有用户间共享数据|
|response|HttpServletResponse|响应对象|
|page|Object|当前页面(Servlet)的对象|
|out|JspWriter|输出对象，数据输出到页面上|
|config|ServletConfig|Servlet的配置对象|
|exception|Throwable|异常对象|

# EL表达式
## 作用
替换和简化jsp页面中java代码的编写，语法格式为`${表达式}`
## 如何使用
jsp默认支持el表达式的
### 获取值
1. el表达式只能从域对象中获取值
2. 语法：
	1. ${域名称.键名}：从指定域中获取指定键的值
		* 域名称：
			1. pageScope		--> pageContext
			2. requestScope 	--> request
			3. sessionScope 	--> session
			4. applicationScope --> application（ServletContext）
		- 举例：在request域中存储了name=张三
		- 获取：`${requestScope.name}`

	2. `${键名}`：表示依次从最小的域中查找是否有该键对应的值，直到找到为止。
	3. 获取对象、List集合、Map集合的值
		1. 对象：`${域名称.键名.属性名}`
			* 本质上会去调用对象的getter方法

		2. List集合：`${域名称.键名[索引]}`
		3. Map集合：
			* `${域名称.键名.key名称}`
			* `${域名称.键名["key名称"]}`
3. 隐式对象：
	* el表达式中有11个隐式对象
	* `pageContext`：
		* 获取jsp其他八个内置对象
			* `${pageContext.request.contextPath}`：动态获取虚拟目录

# JSTL
## 概念
JavaServer Pages Tag Library  JSP标准标签库
## 作用
用于简化和替换jsp页面上的java代码
## 使用步骤：
1. 导入jstl相关jar包
2. 引入核心标签库：`<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`
3. 使用标签
## 常用的JSTL标签
1. if:相当于java代码的if语句
	1. 属性：
		* test 必须属性，接受boolean表达式
		* 如果表达式为true，则显示if标签体内容，如果为false，则不显示标签体内容
		* 一般情况下，test属性值会结合el表达式一起使用
	2. 注意：
		* c:if标签没有else情况，想要else情况，则可以在定义一个c:if标签
```jsp
<c:if test="${4==5}">
	4==5显示
</c:if>
```
2. choose:相当于java代码的switch语句
	1. 使用choose标签声明         			相当于switch声明
	2. 使用when标签做判断         			相当于case
	3. 使用otherwise标签做其他情况的声明    	相当于default
```jsp
<c:choose>
        <c:when test="${number == 1}">星期一</c:when>
        <c:when test="${number == 2}">星期二</c:when>
        <c:when test="${number == 3}">星期三</c:when>
        <c:when test="${number == 4}">星期四</c:when>
        <c:when test="${number == 5}">星期五</c:when>
        <c:when test="${number == 6}">星期六</c:when>
        <c:when test="${number == 7}">星期天</c:when>
        <c:otherwise>数字输入有误</c:otherwise>
</c:choose>
```


3. foreach:相当于java代码的for语句
```jsp
// 普通for循环
<c:forEach begin="0" end="${list.size()}" var="i" step="1">
    ${list[i]}<br>
</c:forEach>

// 增强for循环
<c:forEach items="${list}" var="i">
    ${i}<br>
</c:forEach>
```


