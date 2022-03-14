---
title: mybatis中的延迟加载策略
date: 2020-06-04 21:04:06
tags:
- java
- mybatis
---

# 什么是延迟加载？
就是在需要用到数据时才进行加载，不需要用到数据时就不加载数据。延迟加载也称懒加载.

**好处：**

先从单表查询，需要时再从关联表去关联查询，大大提高数据库性能，因为查询单表要比关联查询多张表速度要快。

**坏处：**

因为只有当需要用到数据时，才会进行数据库查询，这样在大批量数据查询时，因为查询工作也要消耗时间，所以可能造成用户等待时间变长，造成用户体验下降。

# 使用 assocation 实现延迟加载（一对一）
**需求**

查询账户信息同时查询用户信息。

**用户实体类**
```java
public class User implements Serializable {
	private Integer id;
	private String username;
	private Date birthday;
	private String sex;
	private String address;

	// getter and setter...
}

```
**账户实体类**
```java
public class Account implements Serializable {
	private Integer id;
	private Integer uid;
    	private Double money;
	private User user;   // 一个账户对应一个用户信息

	// getter and setter...
}
```
**账户的持久层 DAO 接口**
```java
public interface IAccountDao {
	/**
	* 查询所有账户，同时获取账户的所属用户名称以及它的地址信息
	* @return
	*/
	List<Account> findAll();
}
```
**用户的持久层 DAO 接口**
```java
public interface IUserDao {
	/**
	* 根据 id 查询
	* @param userId
	* @return
	*/
	User findById(Integer id);
}
```

**账户的持久层映射文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.IAccountDao">
    <resultMap id="accountUserMap" type="account">
        <id property="id" column="id"/>
        <result property="uid" column="uid"/>
        <result property="money" column="money"/>
        <association property="user" column="uid" javaType="user" select="com.itheima.dao.IUserDao.findById"/>
    </resultMap>

    <select id="findAll" resultMap="accountUserMap">
        select * from account
    </select>
</mapper>
```
- `association`:
	- `column`：数据库中的列名，或者是列的别名，将作为`IUserDao.java`中`findById`方法的形参
	- `select`：用于加载复杂类型属性的映射语句的 ID，它会从 column 属性指定的列中检索数据，作为参数传递给目标 select 语句

**用户的持久层映射文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.IUserDao">
    <select id="findById" parameterType="java.lang.Integer" resultType="com.itheima.domain.User">
        select * from user where id=#{id}
    </select>
</mapper>
```
**修改mybatis配置文件，开启 Mybatis 的延迟加载策略**
```xml
<settings>
	<setting name="lazyLoadingEnabled" value="true"/>
	<!--mybatis版本>3.4.1可省略，它默认值就是false-->
	<setting name="aggressiveLazyLoading" value="false"/>
</settings>
```
**测试**
```java
@Test
public void testFindAll() {
	// 此时只会查询账户信息，当你遍历list使用到用户信息时mybatis才会再次查询
	List<Account> accounts = accountDao.findAll();
}
```
# 使用 Collection 实现延迟加载(一对多)
**需求：**

完成加载用户对象时，查询该用户所拥有的账户信息。

**在 User 实体类中加入 List<Account>属性**
```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    private List<Account> accounts;
    // get set 方法以省略
}
```
**在账户持久层接口中添加方法**
```java
public interface IAccountDao {
	/**
	* 根据用户 id 查询账户信息
	* @param uid
	* @return
	*/
	List<Account> findByUid(Integer uid);
}
```
**在用户持久层接口中添加方法**
```java
public interface IUserDao {
	/**
	*  查询所有用户，同时获取出每个用户下的所有账户信息
	* @return
	*/
    	List<User> findAll();
}
```
**账户的持久层映射文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.IAccountDao">
    <select id="findAccountByUid" parameterType="int" resultType="account">
        select * from account where uid=#{id}
    </select>
</mapper>
```
**用户持久层映射文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.IUserDao">
    <resultMap id="userAccountMap" type="user">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="address" column="address"/>
        <result property="sex" column="sex"/>
        <result property="birthday" column="birthday"/>
        <collection property="accounts" ofType="account" select="com.itheima.dao.IAccountDao.findAccountByUid" column="id">
        </collection>
    </resultMap>

    <select id="findAll" resultMap="userAccountMap">
        select * from user
    </select>
</mapper>
```
**测试只加载用户信息**
```java
@Test
public void testFindAll() {
        List<User> users = userDao.findAll();
//        for (User user : users) {
//            System.out.println("----------------");
//            System.out.println(user);
//            System.out.println(user.getAccounts());
//        }
}
```