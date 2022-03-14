---
title: spring MVC快速入门
date: 2020-06-10 18:27:09
tags:
- java
- spring mvc
---

# 入门案例
## 创建WEB工程，引入开发的jar包
```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.6.RELEASE</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/javax/javaee-web-api -->
    <dependency>
      <groupId>javax</groupId>
      <artifactId>javaee-web-api</artifactId>
      <version>7.0</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
```
# 配置web.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!-- SpringMVC的核心控制器 -->
    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置Servlet的初始化参数，读取springmvc的配置文件，创建spring容器 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:app-context.xml</param-value>
        </init-param>
        <!-- 配置servlet启动时加载对象 -->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <!-- 配置过滤器，解决中文乱码的问题 -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!-- 指定字符集 -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
## 编写springmvc.xml配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 配置spring创建容器时要扫描的包 -->
    <context:component-scan base-package="com.itcast"/>
    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/page/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!--配置spring开启注解mvc的支持-->
    <mvc:annotation-driven/>
    <!-- 设置静态资源不过滤，以下方式二选一即可 -->
    <mvc:default-servlet-handler/>
    <!--或者-->
    <mvc:resources location="/css/" mapping="/css/**"/> <!-- 样式 -->
    <mvc:resources location="/images/" mapping="/images/**"/> <!-- 图片 -->
    <mvc:resources location="/js/" mapping="/js/**"/> <!-- javascript -->
</beans>
```
## 编写控制器类
```java
@Controller
public class HelloController {
    @RequestMapping("/")
    public String hello(){
        // 它会在WEB-INF/page文件夹下找到index.jsp并返回给浏览器
        return "index";
    }
}
```
`@RequestMapping`：作用是建立请求URL和处理方法之间的对应关系
- 属性：
	- `path`：指定请求路径的url
	- `value`：和path属性是一样的
	- `mthod`：指定该方法的请求方式
	- `params`：指定限制请求参数的条件
	- `headers`：发送的请求中必须包含的请求头
- 细节：
	- 作用在类上：第一级的访问目录
	- 作用在方法上：第二级的访问目录

# 入门案例的执行流程
1. 当启动Tomcat服务器的时候，因为配置了load-on-startup标签，所以会创建DispatcherServlet对象，
就会加载springmvc.xml配置文件
2. 开启了注解扫描，那么HelloController对象就会被创建
3. 从index.jsp发送请求，请求会先到达DispatcherServlet核心控制器，根据配置@RequestMapping注解找到执行的具体方法
4. 根据执行方法的返回值，再根据配置的视图解析器，去指定的目录下查找指定名称的JSP文件
5. Tomcat服务器渲染页面，做出响应

