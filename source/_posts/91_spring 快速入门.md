---
title: spring 快速入门
tags: spring
abbrlink: fdbeaf37
date: 2020-06-08 15:11:22
---

# 新建一个空的maven项目，在pom.xml中添加相关坐标
```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context</artifactId>
	<version>5.2.6.RELEASE</version>
</dependency>
```
# 新建一个类，用于被spring创建
```java
public class Hello {
    public void sayHello(){
        System.out.println("hello wrold!");
    }
}
```
# 创建spring配置文件
在`resources`目录下新建一个文件取名为`ApplicationContext.xml`,并添加一下内容：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

        <!-- 通过spring来创建实体类 -->
	<bean id="hello" class="com.itheima.Hello"></bean>
</beans>
```
添加xml约束在`idea`中会有标签提示哦！
# 编写测试类
```java
public class Test {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("ApplicationContext.xml");
        // 从spring容器中获取Hello对象
        Hello hello = (Hello) applicationContext.getBean("hello");
        hello.sayHello();
    }
}
```