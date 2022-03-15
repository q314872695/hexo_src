---
title: Servlet+spring 纯java配置(无xml)
tags:
  - servlet
  - spring
abbrlink: '6696e004'
date: 2020-06-27 21:53:26
---

# pom.xml
```xml
<!--spring-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.2.6.RELEASE</version>
    </dependency>
    <!--spring整合junit-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.2.6.RELEASE</version>
      <scope>test</scope>
    </dependency>
    <!--spring整合web环境-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.2.6.RELEASE</version>
    </dependency>
<!--java工具类，SpringUtils-->
    <dependency>
      <groupId>cn.hutool</groupId>
      <artifactId>hutool-all</artifactId>
      <version>5.3.8</version>
    </dependency>
```
# 配置类
**MyWebApplicationInitializer.java**
```java
// 等价于web.xml配置文件
public class MyWebApplicationInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(SpringConfig.class);
        ac.refresh();
        StaticLog.info("初始化spring容器");
    }
}
```
**spring配置类**
```java
// spring配置类
@Configuration
@ComponentScan(value = {"cn.itcast","cn.hutool.extra.spring"}) //cn.itcast 自己程序的包，
public class SpringConfig {

}
```
# 使用
```java
Demo2 testDemo = SpringUtil.getBean("testDemo");
```