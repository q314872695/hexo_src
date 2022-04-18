---
title: Filter：过滤器总结
tags: filter
abbrlink: 8d777cfc
date: 2020-05-25 11:04:56
---

# 概念
- web中的过滤器：当访问服务器的资源时，过滤器可以将请求拦截下来，完成一些特殊的功能。
* 过滤器的作用：一般用于完成通用的操作。如：登录验证、统一编码处理、敏感字符过滤...
# 快速入门
## 使用步骤
1. 定义一个类，实现接口Filter
2. 复写方法
3. 配置拦截路径
	1. web.xml
	2. 注解
## 代码
```java
// 使用注解配置
@WebFilter("/*")
public class FilterDemo implements Filter {
    public void destroy() {
    }

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        System.out.println("filterDemo1被执行了....");
        // 放行
        chain.doFilter(req, resp);
    }

    public void init(FilterConfig config) throws ServletException {

    }

}
```
# 使用细节
## web.xml配置
```xml
<filter>
	<filter-name>demo1</filter-name>
	<filter-class>cn.itcast.web.filter.FilterDemo1</filter-class>
</filter>
<filter-mapping>
	<filter-name>demo1</filter-name>
	<!-- 拦截路径 -->
	<url-pattern>/*</url-pattern>
</filter-mapping>
```
## 过滤器执行流程
1. 执行过滤器
2. 执行放行后的资源
3. 回来执行过滤器放行代码下边的代码

## 过滤器生命周期方法
1. `init()`:在服务器启动后，会创建Filter对象，然后调用init方法。只执行一次。用于加载资源
2. `doFilter()`:每一次请求被拦截资源时，会执行。执行多次
3. `destroy()`:在服务器关闭后，Filter对象被销毁。如果服务器是正常关闭，则会执行`destroy()`方法。只执行一次。用于释放资源

## 过滤器配置详解
* 拦截路径配置：
	1. 具体资源路径：`/index.jsp`只有访问index.jsp资源时，过滤器才会被执行
	2. 拦截目录：`/user/*`访问`/user`下的所有资源时，过滤器都会被执行
	3. 后缀名拦截：`*.jsp`访问所有后缀名为jsp资源时，过滤器都会被执行
	4. 拦截所有资源：`/*`访问所有资源时，过滤器都会被执行
* 拦截方式配置：资源被访问的方式
	* 注解配置：
		* 设置`dispatcherTypes`属性
			1. `DispatcherType.REQUEST`：默认值。浏览器直接请求资源
			2. `DispatcherType.FORWARD`：转发访问资源，只有在服务器内部重定向时才会起作用
			3. `DispatcherType.INCLUDE`：包含访问资源
			4. `DispatcherType.ERROR`：错误跳转资源
			5. `DispatcherType.ASYNC`：异步访问资源
	* web.xml配置
		* 设置`<dispatcher></dispatcher>`标签即可
 ## 过滤器链(配置多个过滤器)
* 执行顺序：如果有两个过滤器：过滤器1和过滤器2
	1. 过滤器1
	2. 过滤器2
	3. 资源执行
	4. 过滤器2
	5. 过滤器1 

* 过滤器先后顺序问题：
	1. 注解配置：按照类名的字符串比较规则比较，值小的先执行
		* 如： AFilter 和 BFilter，AFilter就先执行了。
	2. web.xml配置：`<filter-mapping>`谁定义在上边，谁先执行



