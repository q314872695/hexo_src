---
title: mybatis增删改查的简单使用
date: '2020-05-31 22:39'
tags: mybatis
abbrlink: 7ed322fb
---

# 基于代理 Dao 实现 CRUD 操作
**要求：**
1. 持久层接口和持久层接口的映射配置必须在相同的包下
2. 持久层映射配置中 `mapper` 标签的 `namespace` 属性取值必须是持久层接口的全限定类名
3. SQL 语句的配置标签`<select>`,`<insert>`,`<delete>`,`<update>`的 id 属性必须和持久层接口的方法名相同。

## 根据ID查询
**在持久层IUserDao接口中添加 findById 方法**

```java
/**
* 根据 id 查询
* @param userId
* @return
*/
User findById(Integer id);
```

**在用户的映射配置文件`IUserDao.xml`中配置**
```xml
<select id="findById" parameterType="java.lang.Integer" resultType="com.itheima.domain.User">
        select * from user where id=#{id}
</select>
```
细节：
- `resultType` 属性：用于指定结果集的类型(此时java对象的属性名必须和表中的列名一致)。
- `parameterType` 属性：用于指定传入参数的类型。
- sql 语句中使用`#{}`字符：它代表占位符，相当于原来 jdbc 部分所学的?，都是用于执行语句时替换实际的数据。具体的数据是由#{}里面的内容决定的。
- `#{}`中内容的写法：由于数据类型是基本类型，所以此处可以随意写。
## 保存操作
**在持久层IUserDao接口中添加 saveUser 方法**
```java
/**
* 保存用户
* @param user
* @return 影响数据库记录的行数
*/
int saveUser(User user);
```
**在用户的映射配置文件`IUserDao.xml`中配置**
```xml
<insert id="saveUser" parameterType="com.itheima.domain.User">
	insert into user(username,birthday,sex,address) values (#{username},#{birthday},#{sex},#{address})
</insert>
```
**添加测试类中的测试方法**
```java
@Test
public void testSave() {
	IUserDao userDao = session.getMapper(IUserDao.class);
        User user = new User();
        user.setUsername("张三");
        user.setSex("男");
        user.setBirthday(new Date());
        user.setAddress("山西");
        System.out.println(user);
        userDao.saveUser(user);
	// 执行完增，删，更新操作时，需要手动提交事务
        session.commit();
        System.out.println(user);
}
```
**扩展：增加后返回新增用户的id(自动增长的主键)**
```xml
<insert id="saveUser" parameterType="com.itheima.domain.User">
	<!-- 配置保存时获取插入的 id -->
	<selectKey keyProperty="id" resultType="int" keyColumn="id" order="AFTER">
            select last_insert_id()
        </selectKey>
	insert into user(username,birthday,sex,address) values(#{username},#{birthday},#{sex},#{address})
</insert>
或者这样
<insert id="saveUser" parameterType="com.itheima.domain.User" useGeneratedKeys="true" keyProperty="id">
        insert into user(username,birthday,sex,address) values (#{username},#{birthday},#{sex},#{address})
</insert>
```
- `useGeneratedKeys`：首先你的数据库得支持自动生成主键列，比如 MySQL 和 SQL Server），MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键，默认值：false。
- `keyProperty`：（仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（unset）。如果生成列不止一个，可以用逗号分隔多个属性名称。
- `keyColumn`：（仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。
- `order`：可以设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它首先会生成主键，设置 keyProperty 再执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 中的语句
## 用户更新
**在持久层接口中添加更新方法**
```java
/**
* 更新用户
* @param user
* @return 影响数据库记录的行数
*/
int updateUser(User user);
```
**在用户的映射配置文件中配置**
```xml
<update id="updateUser" parameterType="com.itheima.domain.User">
	update user set username=#{username},birthday=#{birthday},sex=#{sex},address=#{address} where id=#{id}
</update>
```
**注意：执行完方法后要手动提交事务**
## 用户删除
**在持久层接口中添加删除方法**
```java
/**
* 根据 id 删除用户
* @param userId
* @return
*/
int deleteUser(Integer userId);
```
**在用户的映射配置文件中配置**
```xml
<delete id="deleteUser" parameterType="java.lang.Integer">
	delete from user where id = #{uid}
</delete>
```
**注意：执行完方法后要手动提交事务**
## 用户模糊查询
**在持久层接口中添加模糊查询方法**
```java
/**
* 根据名称模糊查询
* @param username
* @return
*/
List<User> findByName(String username);
```
**在用户的映射配置文件中配置**
```xml
<select id="findByName" resultType="com.itheima.domain.User" parameterType="java.lang.String">
 	select * from user where username like #{username}
</select>
```
**调用模糊查询方法，需要在查询字符串两侧添加%%**
```java
List<User> users = userDao.findByName("%王%");
for(User user : users){
	System.out.println(user);
}
```
## 查询使用聚合函数
**在持久层接口中添加模糊查询方法**
```java
/**
* 查询总记录条数
* @return
*/
int findTotal();
```
**在用户的映射配置文件中配置**
```xml
<select id="findTotal" resultType="java.lang.Integer">
	select count(*) from user
</select>
```
