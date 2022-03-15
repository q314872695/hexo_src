---
title: spring MVC中的异常处理
tags:
  - java
  - spring mvc
abbrlink: 88f507dc
date: 2020-06-11 21:48:44
---

> 发生异常是不可避免的，希望在发生异常时跳转到一个友好页面
# 步骤
1. 准备要跳转的error.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>错误页面</title>
</head>
<body>
页面发生错误了，请联系管理员。<br>
错误信息：${errmsg}
</body>
</html>
```

2.  自定义异常处理器
```java
/**
 * 自定义异常处理器需要实现HandlerExceptionResolver接口
 */
public class SysExceptionResolver implements HandlerExceptionResolver {

    /**
     * 跳转到具体的错误页面的方法
     */
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        ModelAndView modelAndView = new ModelAndView();
        // 获取发生异常的信息
        modelAndView.addObject("errmsg", ex.getMessage());
        // 要跳转的错误页面
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```
3. 配置异常处理器
把刚才定义的异常处理器加入到springmvc容器中，id没有特别要求随便起

```xml
<!-- 配置异常处理器 --> 
<bean id="sysExceptionResolver" class="cn.itcast.exception.SysExceptionResolver"/>
```
4. 编写控制器测试
```java
@RequestMapping("/exception")
public String testException(){
        // 产生一个异常
        int i=1/0;
        return "success";
}
```
5. 结果
![image.png](https://halo-1257208482.image.myqcloud.com/image_1591883306014.png!webp)




