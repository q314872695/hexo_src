---
title: spring IOC常用注解按作用分类
tags:
  - java
  - spring
abbrlink: 6bbf23e0
date: 2020-06-08 16:48:31
---

> 要想使用注解需要在配置文件中添加注解扫描，并指定扫描的包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

	<!--告诉spring在创建容器时，要扫描的包-->
	<context:component-scan base-package="com.itheima"/>
</beans>
```
# 用于创建对象的
- `@Component`
	- 作用：把当前类对象存入Spring容器中。
	- 属性：
		- `value`：用于指定bean的id，当我们不写时，默认时当前类名且首字母小写。
- `@Controller`：一般用于mvc中的表现层，功能同`@Component`，只是换了个名字，下面的注解也一样
- `@Service`：一般用于mvc中的业务层
- `@Repository`：一般用于mvc中的持久层
# 用于注入数据的
- `@Autowired`
	- 作用：
		- 自动按照类型注入，只要容器中有唯一的bean对象和注入的类型匹配就可以注入成功。
		- 如果容器中没有匹配的类型就会报错
		- 如果容器中有多个匹配的类型也会报错
	- 出现位置：成员变量上或者方法上
	- 细节：不需要set方法也可以注入

**关于容器中有多个匹配类型的解决办法**
1. 修改被`@Autowired`注解的变量名和要注入bean对象的id相同即可。
2. 或者使用`@Qualifier`注解
	- 作用：按照类型注入的基础上再按照名称注入，给成员变量注入时不能单独使用需要配合`@Autowired`，否则报空指针异常,给方法参数注入时可以。
	- 属性：
		- `value`：用于指定注入bean的id。
3. 或者使用`@Resource`注解
	- 作用：直接按照bean的id注入，可以单独使用。
	- 属性：
		- `name`：用于指定注入bean的id。

- `@Value`
	- 作用：用于注入基本类型和String类型的数据
	- 属性：
		- value：用于指定数据的值，也可以使用spring的SpEl

# 用于改变作用范围的
- `@Scope`：和bean标签中的scope属性功能一样
	- 作用：用于指定bean的作用范围
	- 属性：
		- `value`：指定范围的取值，通常为`singleton`、`prototype`
# 和生命周期相关的
- `@PostConstruct`：用于指定初始化方法，同bean标签中的`init-method`属性
- `@PreDestroy`：用于指定销毁方法，同bean标签中的`destroy-method`属性
