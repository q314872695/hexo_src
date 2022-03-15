---
title: spring MVC中请求参数的总结
tags:
  - java
  - spring mvc
abbrlink: b89fe25e
date: 2020-06-10 19:09:02
---

# 请求参数的绑定
## 请求参数的绑定说明
1. 绑定机制
	1.  表单提交的数据都是k=v格式的 username=haha&password=123
	2. **SpringMVC的参数绑定过程是把表单提交的请求参数，作为控制器中方法的参数进行绑定的**
	3. 要求：**提交表单的name和参数的名称是相同的**
2. 支持的数据类型
	1. 基本数据类型和字符串类型
		- 提交表单的name和参数的名称是相同的且区分大小写
	2. 实体类型（JavaBean）
		-  提交表单的name和JavaBean中的属性名称需要一致
		- 如果一个JavaBean类中包含其他的引用类型，那么表单的name属性需要编写成：对象.属性 例如：
`user.age`
	3. 集合数据类型（List、map集合等）
		- JSP页面编写方式：list[0].属性、map['key'].属性
## 请求参数中文乱码的解决
```xml
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
```
## 在控制器中使用原生的ServletAPI对象
只需要在控制器的方法参数定义`HttpServletRequest`和`HttpServletResponse`对象即可
# 自定义类型转换器
## 需求
请求参数中含有日期，把它保存在javaBean中Date类型的属性中，例如2020/6/10是可以正常转换的，但有时候我们的格式是2020-6-10，他就会报400异常，所以需要自定义一个类型转换器去手动转换它。
## 步骤
### 自定义类型转换器
```java
public class StringToDateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {
        Date date=null;
        try {
            date = new SimpleDateFormat("yyyy-MM-dd").parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```
###  注册自定义类型转换器，在springmvc.xml配置文件中编写配置
```xml
<!--配置自定义类型转换器-->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.itcast.utils.StringToDateConverter"></bean>
            </set>
        </property>
    </bean>
<!--配置spring开启注解mvc的支持-->
<mvc:annotation-driven conversion-service="conversionService"/>
```
# 常用注解
## @RequestParam
- 作用：把请求中的指定名称的参数传递给控制器中的形参赋值
- 属性
	- value：请求参数中的名称
	- required：请求参数中是否必须提供此参数，默认值是true，必须提供
- 示例
```java
@RequestMapping(path="/hello") 
public String sayHello(@RequestParam(value="username",required=false)String name) { 
	System.out.println("aaaa"); 
	System.out.println(name); 
	return "success"; 
}
```
## @RequestBody
- 作用：用于获取请求体的内容（注意：get方法不可以）
- 属性:
	-  required：是否必须有请求体，默认值是true
### 使用@RequestBody获取请求体数据
```java
@RequestMapping("/requestbody")
public String params(@RequestBody String body) {
        System.out.println(body);
        return "success";
}
```
### 使用`@RequestBody`注解把json的字符串转换成JavaBean的对象
- 需要加入jackson 的包。
```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.11.0</version>
</dependency>
```
- 示例
```java
@RequestMapping("/json1")
public String json(@RequestBody User user) {
        System.out.println(user);
        return "success";
}
```


## @PathVariable
- 作用：拥有绑定url中的占位符的。例如：url中有/delete/{id}，{id}就是占位符
- 属性
	- value：指定url中的占位符名称
- 示例
```java
@RequestMapping(path="/hello/{id}") 
public String sayHello(@PathVariable(value="id") String id) { 
	System.out.println(id); 
	return "success"; 
}
```
## @RequestHeader
- 作用：获取指定请求头的值
- 属性
	-  value：请求头的名称
- 示例
```java
@RequestMapping(path="/hello") 
public String sayHello(@RequestHeader(value="Accept") String header) { 
	System.out.println(header); 
	return "success"; 
}
```
## @CookieValue
- 作用：用于获取指定cookie的名称的值
- 属性
	-  value：cookie的名称
- 示例
```java
@RequestMapping(path="/hello") 
public String sayHello(@CookieValue(value="JSESSIONID") String cookieValue) { 
	System.out.println(cookieValue); 
	return "success"; 
}
```
## @ModelAttribute
- 作用
	- 出现在方法上：表示当前方法会在控制器方法执行前线执行。
	- 出现在参数上：获取指定的数据给参数赋值。
- 应用场景
	- 当提交表单数据不是完整的实体数据时，保证没有提交的字段使用数据库原来的数据。
- 示例
修饰的方法有返回值
```java
    @ModelAttribute
    public User showUser(String username) {
        System.out.println("showUser执行了...");
        // 模拟从数据库中查询对象
        User user = new User();
        user.setName(username);
        user.setAge(18);
        return user;
    }

    @RequestMapping(path = "/updateUser")
    public String updateUser(User user) {
        System.out.println(user);
        return "success";
    }
```
修饰的方法没有返回值
```java
@ModelAttribute
public void showUser(String name, Map<String, User> map) {
        System.out.println("showUser执行了...");
        // 模拟从数据库中查询对象
        User user = new User();
        user.setName("哈哈");
        user.setAge(100);
        map.put("abc", user);
}

@RequestMapping(path = "/updateUser")
public String updateUser1(@ModelAttribute(value = "abc") User user) {
        System.out.println(user);
        return "success";
}
```
## @SessionAttributes
- 作用：用于多次执行控制器方法间的参数共享
-  属性
	- value：指定存入属性的名称
- 示例
```java
@Controller
@SessionAttributes({"username", "password", "age"})  // 把数据存到session中
@RequestMapping("/t")
public class HaController {
    // 向session中存入值
    @RequestMapping(path = "/save")
    public String save(Model model) {
        System.out.println("向session域中保存数据");
        // Model 默认存在request域中，当控制器被@SessionAttributes注解的的属性也会在session域中存一份
        model.addAttribute("username", "root");
        model.addAttribute("password", "123");
        model.addAttribute("age", 20);
        return "success";
    }

    // 从session中获取值
    @RequestMapping(path = "/find")
    public String find(ModelMap modelMap) {
        String username = (String) modelMap.get("username");
        String password = (String) modelMap.get("password");
        Integer age = (Integer) modelMap.get("age");
        System.out.println(username + " : " + password + " : " + age);
        return "success";
    }
    // 清除session
    @RequestMapping(path = "/delete")
    public String delete(SessionStatus status) {
        status.setComplete();
        return "success";
    }
}
```
Model默认存储在request域中，当控制器被`@SessionAttributes`修饰后，同时也会在session域中存储一份。

## @SessionAttribute
用与从session域中读取指定名称的值
- 属性：
	- value：要获取的值
- 作用范围：只能修饰形参
- 示例
```java
@RequestMapping(path = "/getsession")
public String save1(@SessionAttribute("username") String name) {
        System.out.println("向session域中读取数据");
        System.out.println(name);
        return "success";
}
```










