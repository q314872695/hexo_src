---
title: spring MVC之文件上传
tags: spring mvc
abbrlink: b1d9f7e1
date: 2020-06-11 14:53:05
---

# 上传到本地服务器
文件上传的方式有两种实现，如果是Servlet3.0以上，则不需要导第三方jar包，否则需要。
## Apache Commons FileUpload
该方式需要导入第三方jar包
### 准备工作
1. 导包
```xml
<dependency>
	<groupId>commons-fileupload</groupId>
	<artifactId>commons-fileupload</artifactId>
	<version>1.4</version>
</dependency>
```
2. 在springmvc.xml配置文件中配置解析器对象
```xml
<!-- 配置文件解析器对象，要求id名称必须是multipartResolver -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="10485760"/>
</bean>
```
### 代码编写
1. 编写文件上传的JSP页面
```jsp
<h3>文件上传</h3> 
<form action="user/fileupload" method="post" enctype="multipart/form-data"> 
选择文件：<input type="file" name="upload"/><br/> 
<input type="submit" value="上传文件"/> 
</form>
```
2. 编写文件上传的控制器
```java
@RequestMapping("/fileupload")
public String upload(HttpServletRequest request, MultipartFile upload) {
        // 获取上传的目录
        String path = request.getServletContext().getRealPath("/upload");
        File file = new File(path);
        // 如果目录不存在就创建
        if (!file.exists()) {
            file.mkdir();
        }
        // 获取文件名
        String fileName = upload.getOriginalFilename();
        try {
            upload.transferTo(new File(file,fileName));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "success";
}
```
**注意：**`MultipartFile`的形参变量必须和`<input type="file" name="upload"/>`中的`name`名称一样，否则无法匹配。
## Servlet 3.0
该方式使用于Servlet3.0以上版本，需要通过Servlet容器配置启用Servlet 3.0多部分解析。
### 环境准备
1. 在web.xml中`DispatcherServlet`的标签中添加`<multipart-config>`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1"
         metadata-complete="true">
    <!-- SpringMVC的核心控制器 -->
    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置Servlet的初始化参数，读取springmvc的配置文件，创建spring容器 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!-- 配置servlet启动时加载对象 -->
        <load-on-startup>1</load-on-startup>
        <!--基于Servlet3.0文件上传-->
        <multipart-config/>
    </servlet>
    ....
</web-app>
```
2. 在springmvc.xml配置文件中配置解析器对象
```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"/>
```
## 代码编写
和上面一样不需要改动
# 跨服务器上传
## 准备一个图片服务器
1. 新建一个java web项目，并修改响应的端口，否则端口被占用报错。
2. 修改tomcat的配置文件web.xml,在`DefaultServlet`的标签中添加
```xml
<init-param>
        <param-name>readonly</param-name>
        <param-value>false</param-value>
</init-param>
```
tomcat默认的情况下是过滤到静态资源的上传的，所以需要我们手动的设置一下
3. web.xml中的`DefaultServlet`标签的完整配置
```xml
<servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        
        <init-param>
       	    <param-name>readonly</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
</servlet>
```
## 在另一个项目中编写控制器代码
导入相关jar包
```xml
<dependency>
	<groupId>com.sun.jersey</groupId>
	<artifactId>jersey-client</artifactId>
	<version>1.18.1</version>
</dependency>
```
编写控制器中的代码
```java
@RequestMapping("/fileupload3")
public String fileupload3(MultipartFile upload) throws Exception {
        System.out.println("SpringMVC跨服务器方式的文件上传...");
        // 定义图片服务器的请求路径
        String path = "http://localhost:8090/uploadServer_war_exploded/upload/";
        // 获取到上传文件的名称
        String filename = upload.getOriginalFilename();
        // 向图片服务器上传文件
        // 创建客户端对象
        Client client = Client.create();
        // 连接图片服务器
        WebResource webResource = client.resource(path + filename);
        // 上传文件
        webResource.put(upload.getBytes());
        return "success";
}
```

