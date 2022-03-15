---
title: spring MVC 中的拦截器配置
tags:
  - java
  - spring mvc
abbrlink: 62f73bbe
date: 2020-06-12 09:57:42
---

# 拦截器概述
1. SpringMVC框架中的拦截器用于对处理器进行预处理和后处理的技术。
2. 可以定义拦截器链，连接器链就是将拦截器按着一定的顺序结成一条链，在访问被拦截的方法时，拦截器链中的拦截器会按着定义的顺序执行。
3. 拦截器和过滤器的功能比较类似，有区别
	1. 过滤器是Servlet规范的一部分，任何框架都可以使用过滤器技术。
	2. 拦截器是SpringMVC框架独有的。
	3. 过滤器配置了/*，可以拦截任何资源。
	4. 拦截器只会对控制器中的方法进行拦截。
4. 拦截器也是AOP思想的一种实现方式
5. 想要自定义拦截器，需要实现HandlerInterceptor接口。
# 自定义拦截器步骤
1.  创建类，实现HandlerInterceptor接口，重写需要的方法
```java
public class MyInterceptor implements HandlerInterceptor {
    /**
     * 预处理，controller执行前执行
     * @return  true:放行;false:拦截
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器执行了");
        // request.getRequestDispatcher("WEB-INF/page/success.jsp").forward(request,response);
        return true;
    }
}
```
2. 在springmvc.xml中配置拦截器类
```xml
<!--配置拦截器-->
<mvc:interceptors>
        <mvc:interceptor>
            <!--要拦截的路径-->
            <mvc:mapping path="/**"/>
            <!--不要拦截的路径-->
            <mvc:exclude-mapping path="/t/**"/>
            <!--配置拦截器对象-->
            <bean class="com.itcast.interceptor.MyInterceptor"/>
        </mvc:interceptor>
</mvc:interceptors>
```
# HandlerInterceptor接口中的方法
1. `preHandle`方法是controller方法执行前拦截的方法
	1. 可以使用request或者response跳转到指定的页面
	2. return true放行，执行下一个拦截器，如果没有拦截器，执行controller中的方法。
	3. return false不放行，不会执行controller中的方法。
	4.  可以使用转发或者重定向直接跳转到指定的页面。
2. `postHandle`是controller方法执行后执行的方法，在JSP视图执行前。
	1. 可以使用request或者response跳转到指定的页面
	2. 如果指定了跳转的页面，那么controller方法跳转的页面将不会显示。
3. `afterCompletion`方法是在JSP执行后执行
	1. request或者response不能再跳转页面了
# 配置多个拦截器
1. 再编写一个拦截器的类
2. 配置2个拦截器
```xml
    <!--配置拦截器-->
    <mvc:interceptors>
        <!--配置第一个拦截器-->
        <mvc:interceptor>
            <!--要拦截的路径-->
            <mvc:mapping path="/**"/>
            <!--不要拦截的路径-->
            <mvc:exclude-mapping path="/t/**"/>
            <!--配置拦截器对象-->
            <bean class="com.itcast.interceptor.MyInterceptor"/>
        </mvc:interceptor>
        <!--配置第二个拦截器-->
        <mvc:interceptor>
            <mvc:mapping path="/t/**"/>
            <bean class="com.itcast.interceptor.MyInterceptor2"/>
        </mvc:interceptor>
    </mvc:interceptors>
```







