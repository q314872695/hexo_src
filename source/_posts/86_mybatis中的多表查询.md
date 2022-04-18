---
title: mybatis中的多表查询
tags: mybatis
abbrlink: c381a747
date: 2020-06-04 15:42:52
---

# 一对一查询（多对一）
**数据库中表的关系图**
![image.png](https://halo-1257208482.image.myqcloud.com/202204051755601.png!webp)

- `user`表和`account`表是一对多关系
- `account`表中`uid`是外键
- `user`相对于`account`表是外键表
> 需求：查询所有account表的信息，关联查询下单用户信息。
> 注意：因为一个账户信息只能供某个用户使用，所以从查询账户信息出发关联查询用户信息为一对一查询。
> 查询的sql：`select u.*,a.id as aid,uid,money from account a join user u on a.uid=u.id` 由于account和user表中的id列重名，所以给acount表id起个别名aid。

**定义实体类**
```java
public class User implements Serializable {
	private Integer id;
	private String username;
	private Date birthday;
	private String sex;
	private String address;

	// getter and setter ..	
}
public class Account implements Serializable {
	private Integer id;
	private Integer uid;
	private Double money;
	// 从account表看每个账户对应一个用户信息，是一对一
	private User user;
	
	// getter and setter ...
}
```
**编写持久层接口**
```java
public interface IAccountDao {
    List<Account> findAll();
}
```
**编写映射文件**
```xml
<mapper namespace="com.itheima.dao.IAccountDao">
    <resultMap id="accountUserMap" type="com.itheima.domain.Account">
        <id property="id" column="aid"></id>
        <result property="uid" column="uid"></result>
        <result property="money" column="money"></result>
        <association property="user" column="uid" javaType="com.itheima.domain.User">
            <id property="id" column="uid"></id>
            <result property="username" column="username"></result>
            <result property="address" column="address"></result>
            <result property="sex" column="sex"></result>
            <result property="birthday" column="birthday"></result>
        </association>
    </resultMap>

    <select id="findAll" resultMap="accountUserMap">
        select u.*,a.id as aid,uid,money from account a join user u on a.uid=u.id
    </select>
</mapper>
```
- `association`标签处理一对一的关系
	- `property`：关联查询的结果存到Account对象的哪个属性上
	- `javaType`：指定这个关联对象的具体类型

**测试**
```java
@Test
public void testFindAll() {
        List<Account> accounts = accountDao.findAll();
        for (Account account : accounts) {
            System.out.println("-------------------");
            System.out.println(account);
            System.out.println(account.getUser());
        }
}
结果：
-------------------
Account{id=1, uid=46, money=1000.0}
User{id=46, username='老王', birthday=Wed Mar 07 17:37:26 CST 2018, sex='女', address='北京'}
-------------------
Account{id=2, uid=45, money=1000.0}
User{id=45, username='传智播客', birthday=Sun Mar 04 12:04:06 CST 2018, sex='男', address='北京金燕龙'}
-------------------
```
# 一对多查询(多对多)
> 需求：查询所有用户信息及用户关联的账户信息。
> 分析：用户信息和他的账户信息为一对多关系，并且查询过程中如果用户没有账户信息，此时也要将用户信息查询出来。

**User 类加入`List<Account>`**
```java
public class User implements Serializable {
	private Integer id;
	private String username;
	private Date birthday;
	private String sex;
	private String address;
	private List<Account> accounts;
	// getter and setter ..	
}
```
**编写持久层接口**
```java
public interface IUserDao {
    List<User> findAll();
}
```
**映射文件配置**
```xml
<mapper namespace="com.itheima.dao.IUserDao">
    <resultMap id="userAccountMap" type="user">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="address" column="address"/>
        <result property="sex" column="sex"/>
        <result property="birthday" column="birthday"/>
        <collection property="accounts" ofType="account">
            <id property="id" column="aid"/>
            <result property="uid" column="uid"/>
            <result property="money" column="money"/>
        </collection>
    </resultMap>
    
    <select id="findAll" resultMap="userAccountMap">
        select u.*,a.id as aid,uid,money from user u left join account a on a.uid=u.id
    </select>
</mapper>
```
- `collection`标签定义了用户关联的账户信息。表示关联查询结果集（一对多）
	- `property`：关联查询的结果集存储在 User 对象上的哪个属性。
	- `ofType`：指定关联查询的结果集中的对象类型即List中的对象类型。此处可以使用别名，也可以使用全限定名

**测试**
```java
@Test
public void testFindAll() {
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println("--------------------------");
            System.out.println(user);
            System.out.println(user.getAccounts());
        }
}
结果：
--------------------------
User{id=46, username='老王', birthday=Wed Mar 07 17:37:26 CST 2018, sex='女', address='北京'}
[Account{id=1, uid=46, money=1000.0}, Account{id=3, uid=46, money=2000.0}]
--------------------------
User{id=45, username='传智播客', birthday=Sun Mar 04 12:04:06 CST 2018, sex='男', address='北京金燕龙'}
[Account{id=2, uid=45, money=1000.0}]
--------------------------
User{id=41, username='老王', birthday=Tue Feb 27 17:47:08 CST 2018, sex='男', address='北京'}
[]
--------------------------
User{id=42, username='小二王', birthday=Fri Mar 02 15:09:37 CST 2018, sex='女', address='北京金燕龙'}
[]
```
**关于多对多查询**
教师和班级是我们常见的多对多关系。
从班级角度看，一个班级可以被多个教师教（一对多）；
从教师角度看，一个教师又可以教多个班级（一对多）。
其实，多对多是可以看成两个一多对关系的。