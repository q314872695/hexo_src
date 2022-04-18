---
title: spring MVC中的响应数据总结
tags: spring mvc
abbrlink: 1c068cee
date: 2020-06-11 10:45:11
---

# 返回值分类
## 返回字符串
Controller方法返回字符串可以指定逻辑视图的名称，根据视图解析器为物理视图的地址。
```java
@RequestMapping(value="/hello") 
public String sayHello() { 
	System.out.println("Hello SpringMVC!!"); 
	// 跳转到XX页面 return "success"; 
}
```
## 返回值是void
- 如果控制器的方法返回值编写成void，默认会跳转到`@RequestMapping(value="/hello")` `hello.jsp`的页面。如果找不到程序报404的异常，表示查找JSP页面没有找到。

## 返回值是ModelAndView对象
- ModelAndView对象是Spring提供的一个对象，可以用来调整具体的JSP视图
```java
@RequestMapping("/model")
    public ModelAndView modelAndView(){
        ModelAndView modelAndView = new ModelAndView();
        User user = new User();
        user.setName("张三");
        user.setAge(18);
        // user对象会被存入request域中
        modelAndView.addObject("user", user);
        // 跳转到success.jsp的页面
        modelAndView.setViewName("success");
        return modelAndView;
    }
```
# SpringMVC框架提供的转发和重定向
## forward请求转发
controller方法返回String类型，想进行请求转发也可以编写成
```java
    //使用forward关键字进行请求转发
    @RequestMapping("/delete")
    public String delete() throws Exception {
        System.out.println("delete方法执行了...");
        //不走视图解析器了，所以需要编写完整的路径
        return "forward:/WEB-INF/page/success.jsp";
    }
```
## redirect重定向
controller方法返回String类型，想进行重定向也可以编写成
```java
    //使用redirect关键字进行重定向
    @RequestMapping("/test")
    public String test(){
        System.out.println("delete方法执行了...");
	// 不需要加虚拟目录，springmvc会自动帮我们加上
        return "redirect:/user/model";
    }
```
# @ResponseBody注解使用
## 响应字符串
```java
@RequestMapping("/string")
@ResponseBody
public String string() {	// 返回值是字符串
	return "Hello world!";
}
```
## 把JavaBean对象转换成json字符串，直接响应
- 需要导入jackjson包
```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.11.0</version>
</dependency>
```
- 示例
```java
@RequestMapping("/json2")
@ResponseBody	
public Object jso2() {	// 返回值是Object类型
        User user = new User();
        user.setName("张三");
        user.setAge(18);
        user.setDate(new Date());
        return user;
}
```



