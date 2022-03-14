---
title: mybatis中的SqlMapConfig.xml配置文件
date: 2020-06-02 15:19:29
tags:
- java
- mybatis
---

> 首先呢这个主配置文件的名字是可以随便起的，想叫什么就叫什么
# 配置文件结构
- configuration（配置）
- properties（属性）
- settings（设置）
- typeAliases（类型别名）
- typeHandlers（类型处理器）
- objectFactory（对象工厂）
- plugins（插件）
- environments（环境配置）
	- environment（环境变量）
		- transactionManager（事务管理器）
		- dataSource（数据源）
- databaseIdProvider（数据库厂商标识）
- mappers（映射器）

# 属性（properties）
在使用 properties 标签配置时，我们可以采用两种方式指定属性配置。
## 第一种
```xml
<properties>
	<property name="jdbc.driver" value="com.mysql.jdbc.Driver"/>
	<property name="jdbc.url" value="jdbc:mysql://localhost:3306/eesy"/>
	<property name="jdbc.username" value="root"/>
	<property name="jdbc.password" value="1234"/>
</properties>
```
## 第二种
**在 resources 下定义 jdbcConfig.properties 文件（maven项目）**
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.username=root
jdbc.password=123456
```
**properties 标签配置**
```xml
<properties resource="jdbcConfig.properties"></properties>
```
- `resource`属性：用于指定 properties 配置文件的位置，要求配置文件必须在类路径下。
- 或者也可以使用`url`属性用来获取网络中的配置文件，格式：`<properties url="..."></properties>`

## 此时 dataSource 标签就变成了引用上面的配置
```xml
<dataSource type="POOLED">
	<property name="driver" value="${jdbc.driver}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</dataSource>
```
# 类型别名（typeAliases）
类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。例如：
```xml
<typeAliases>
  <typeAlias alias="user" type="com.itheima.domain.User"/>
</typeAliases>
```
或者也可以批量别名定义
```xml
<typeAliases>
	<!-- 批量别名定义，扫描整个包下的类，别名为类名（首字母大写或小写都可以） -->
	<package name="com.itheima.domain"/>
</typeAliases>
```

# 映射器（mappers）
我们要告诉 MyBatis 到哪里去找映射文件
- 使用相对于类路径的资源如：
```xml
<mappers>
        <mapper resource="com/itheima/dao/IUserDao.xml"/>
</mappers>
```
- 使用 mapper 接口类路径(基于注解开发)
```xml
<mappers>
	<mapper class="com.itheima.dao.IUserDao"/>
</mappers>
```
- 注册指定包下的所有 mapper 接口
	- 此种方法要求 mapper 接口名称和 mapper 映射文件名称相同，且放在同一个目录中。
	- xml和注解形式通用，批量注册
```xml
<mappers>
	<package name="com.itheima.dao"/>
</mappers>
```



