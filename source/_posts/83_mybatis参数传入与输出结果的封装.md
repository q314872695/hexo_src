---
title: mybatis参数传入与输出结果的封装
tags: mybatis
abbrlink: 523fb24f
date: 2020-06-02 11:49:20
---

#  Mybatis 的参数传入
## parameterType 配置参数
- 基本类型和String我们可以直接写类型名称(别名)，也可以使用包名.类名的方 式，例如：`java.lang.String`。
- 实体类类型，目前我们只能使用全限定类名。
- 究其原因，是 mybaits 在加载时已经把常用的数据类型注册了别名，从而我们在使用时可以不写包名，而我们的是实体类并没有注册别名，所以必须写全限定类名。


**mybatis中支持的默认别名：**

下面是一些为常见的 Java 类型内建的类型别名。它们都是不区分大小写的，注意，为了应对原始类型的命名重复，采取了特殊的命名风格。

|别名|映射的类型|
|-------|-------|
|_byte|byte|
|_long	|long
|_short	|short
|_int	|int
|_integer|	int
|_double|	double
|_float	|float
|_boolean|boolean
|string	|String
|byte	|Byte
|long	|Long
|short	|Short
|int	|Integer
|integer|	Integer
|double	|Double
|float	|Float
|boolean|	Boolean
|date	|Date
|decimal|	BigDecimal
|bigdecimal|	BigDecimal
|object	|Object
|map	|Map
|hashmap|	HashMap
|list	|List
|arraylist|	ArrayList
|collection|	Collection
|iterator|	Iterator

## MyBatis传递多个参数
### 1. 通过JavaBean传递多个参数

**编写 QueryVo.java**

```java
// 查询条件对象
public class QueryVo implements Serializable {
	private User user;
	public User getUser() {
		return user; 
	}
	public void setUser(User user) {
		this.user = user; 
	} 
}
```

**编写持久层接口**

```java
public interface IUserDao {
	/**
	* 根据 QueryVo 中的条件查询用户
	* @param vo
	* @return
	*/
	List<User> findByVo(QueryVo vo);
}
```

**持久层接口的映射文件**

```xml
<!-- 根据用户名称模糊查询，参数变成一个 QueryVo 对象了 -->
<select id="findByVo" resultType="com.itheima.domain.User" parameterType="com.itheima.domain.QueryVo">
	select * from user where username like #{user.username};
</select>
```
### 2. 使用map接口传递参数
**编写持久层接口**
```java
// 根据姓名和地址查找用户
List<User> findByNameAndAddress(Map map);
```
**持久层接口的映射文件**
```xml
<select id="findByNameAndAddress" parameterType="map" resultType="com.itheima.domain.User">
        select * from user where username like concat('%',#{name},'%') and address like concat('%',#{address},'%')
</select>
```

**测试类**

```java
@Test
public void test1(){
	HashMap<String, String> map = new HashMap<String, String>();
        map.put("name", "张三");
        map.put("address", "山西");
        List<User> users = userDao.findByNameAndAddress(map);
        for (User user : users) {
            System.out.println(user);
        }
}
```
### 3. 使用注解传递多个参数
- `@Param`：如果你的映射方法接受多个参数，就可以使用这个注解自定义每个参数的名字。如果使用了 @Param("person")，参数就会被命名为 #{person}。

**编写持久层接口**
```java
// 根据姓名和地址查找用户
List<User> findByNameAndAddress1(@Param("username")String username,@Param("address")String address);
```
**持久层接口的映射文件**
```xml
<select id="findByNameAndAddress1"  resultType="com.itheima.domain.User">
        select * from user where username like concat('%',#{username},'%') and address like concat('%',#{address},'%')
</select>
```

**测试类**

```java
@Test
public void test1(){
        List<User> users = userDao.findByNameAndAddress1("张三","山");
        for (User user : users) {
            System.out.println(user);
        }
}
```

#  Mybatis 的输出结果封装
## resultType 配置结果类型
- `resultType`属性可以指定结果集的类型，它支持基本类型和实体类类型。
- 需要注意的是，它和 `parameterType`一样，如果注册过类型别名的，可以直接使用别名。没有注册过的必须使用全限定类名。
- 实体类中的属性名称必须和查询语句中的列名保持一致，否则无法
实现封装

### 属性名和列名称不一致的解决办法：
1. 给查询结果的列名称起别名

**实体类**
```java
public class User implements Serializable {
	private Integer userId;
	private String userName;
	private Date userBirthday;
	private String userSex;
	private String userAddress;

	// ...getter and  setter
}
```
**映射文件**
```xml
<!-- 配置查询所有操作 -->
<select id="findAll" resultType="com.itheima.domain.User">
	select id as userId,username as userName,birthday as userBirthday,sex as userSex,address as userAddress from user
</select>
```

2. 使用`resultMap`标签
- `resultMap`标签可以建立查询的列名和实体类的属性名称不一致时建立对应关系。从而实现封装。
- 同时 `resultMap`可以实现将查询结果映射为复杂类型的javaBean对象，一对多，多对多查询时用到

**配置映射文件**
```xml
<!-- 建立 User 实体和数据库表的对应关系
	type 属性：指定实体类的全限定类名
	id 属性：给定一个唯一标识，是给查询 select 标签引用用的。
-->
<resultMap type="com.itheima.domain.User" id="userMap">
	<id column="id" property="userId"/>
	<result column="username" property="userName"/>
	<result column="sex" property="userSex"/>
	<result column="address" property="userAddress"/>
	<result column="birthday" property="userBirthday"/>
</resultMap>

<!-- 配置查询所有操作 --> 
<select id="findAll" resultMap="userMap">
	select * from user
</select>
```

- `id`标签：用于指定主键字段
- `result`标签：用于指定非主键字段
- `column`属性：用于指定数据库列名
- `property`属性：用于指定实体类属性名称

