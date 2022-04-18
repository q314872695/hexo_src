---
title: mybatis基于注解开发实例
tags: mybatis
abbrlink: 1c229434
date: 2020-06-07 09:37:53
---

# 常用注解说明

|注解|使用对象|XML 等价形式|描述 |
|-------|-------|-------|------|
|`@Insert`<br/>`@Update`<br/>`@Delete`<br/>`@Select`|方法|`<insert>`<br/>`<update>`<br/>`<delete>`<br/>`<select>`<br/>|每个注解分别代表将会被执行的 SQL 语句。|
|`@InsertProvider`<br/>`@UpdateProvider`<br/>`@DeleteProvider`<br/>`@SelectProvider`|方法|`<insert>`<br/>`<update>`<br/>`<delete>`<br/>`<select>`<br/>|允许构建动态 SQL。这些备选的 SQL 注解允许你指定返回 SQL 语句的类和方法，以供运行时执行。|
|`@Result`|N/A|`<result>`<br/>`<id>`|在列和属性或字段之间的单个结果映射。属性：`id`、`column`、`javaType`、`jdbcType`、`typeHandler`、`one`、`many`。|
|`@Results`|方法|`<resultMap>`|一组结果映射，指定了对某个特定结果列，映射到某个属性或字段的方式。属性：value、id。value 属性是一个 Result 注解的数组。而 id 属性则是结果映射的名称。|
|`@One`|N/A|`<association>`|复杂类型的单个属性映射。|
|`@Many`|N/A|`<collection>`|复杂类型的集合属性映射。|
|`@CacheNamespace`|类|`<cache>`|为给定的命名空间（比如类）配置缓存。|
|`@ResultMap`|方法|N/A|实现引用@Results 定义的封装|

# 使用 Mybatis 注解实现基本 CRUD
**编写实体类**
```java
// 和数据库列名一致的实体类
public class User implements Serializable {
    private Integer id;
    private String username;
    private String sex;
    private String address;
    private Date birthday;
    // get set方法以省略
}
// 和数据列名不一致的实体类
public class User1 implements Serializable {
    private Integer uid;
    private String uname;
    private String usex;
    private String uaddress;
    private Date ubirthday;
    // get set方法以省略
}
```
**使用注解方式开发持久层接口**
```java
public interface IUserDao {
    // 和数据库列名一致的情况
    @Select("select * from user")
    List<User> findAll();

    // 和数据库列名不一致的情况
    @Select("select * from user")
    @Results(id = "userMap",value = {
            @Result(id = true,property = "uid",column = "id"),
            @Result(property = "uname",column = "username"),
            @Result(property = "usex",column = "sex"),
            @Result(property = "uaddress",column = "address"),
            @Result(property = "ubirthday",column = "birthday")
    })
    List<User1> findAll1();

    /**
     * 根据id 查询一个用户
     * @param userId
     * @return
     */
    @Select("select * from user where id = #{uid} ")
    @ResultMap("userMap") //引用@Result定义的封装
    User1 findById(Integer userId);

    @Insert("insert into user(username,address,sex,birthday) values(#{uname},#{uaddress},#{usex},#{ubirthday})")
    @Options(useGeneratedKeys = true, keyColumn = "id",keyProperty = "uid") //添加用户并返回主键id
    void saveUser(User1 user);
}
```
# 使用注解实现一对一复杂关系映射及延迟加载
**需求：**

加载账户信息时并且加载该账户的用户信息，根据情况可实现延迟加载。

**实体类**
```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private String sex;
    private String address;
    private Date birthday;
    private List<Account> accounts; // 一个用户对应多个账户信息
    // getter setter 方法以省略
}
public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private Double money;
    private User user;  //一个账户对应一个用户信息
    // getter setter 方法以省略
}
```

**添加用户的持久层接口并使用注解配置**
```java
public interface IUserDao {
    /**
     * 根据id 查询一个用户
     * @param userId
     * @return
     */
    @Select("select * from user where id = #{uid} ")
    @Results({
            @Result(id = true,property = "uid",column = "id"),
            @Result(property = "uname",column = "username"),
            @Result(property = "usex",column = "sex"),
            @Result(property = "uaddress",column = "address"),
            @Result(property = "ubirthday",column = "birthday")
    })
    User1 findById(Integer userId);
}
```


**添加账户的持久层接口并使用注解配置**
```java
public interface IAccountDao {
    /**
     * 查询所有账户，采用延迟加载的方式查询账户的所属用户
     *
     * @return
     */
    @Select("select * from account")
    @Results(id = "accountMap",
            value = {
                    @Result(id = true, column = "id", property = "id"),
                    @Result(column = "uid", property = "uid"),
                    @Result(column = "money", property = "money"),
                    @Result(column = "uid", property = "user",
                            one = @One(select = "com.itheima.dao.IUserDao.findById",fetchType = FetchType.LAZY))
            })  // 取消延迟加载把fetchType = FetchType.LAZY去掉即可
    List<Account> findAll();
}
```
# 使用注解实现一对多复杂关系映射
和一对一关系映射类似
- 在User类中添加Account属性
- 在IAccountDao接口中添加根据id查找账户的方法
- 在IUserDao接口中添加查找全部用户的方法并配置相关注解
- 注意这次使用的的many属性和@Many注解，而不是one

# 使用注解构造动态SQL
**需求**
根据username和sex是否为空，查询时动态构造sql语句

**创建个sql类用于定义动态sql**
```java
public class MySql {
    public String findByNameAndAddress(final User user) {
        return new SQL(){{
            SELECT("*");
            FROM("user");
            if (user.getUsername() != null) {
                WHERE("username like #{username}");
            }
            if (user.getSex() != null) {
                WHERE("sex like #{sex}");
            }
        }}.toString();
    }
}
```
**修改IUserDao接口**
```java
public interface IUserDao {
    @SelectProvider(type = MySql.class,method = "findByNameAndAddress")
    List<User> findByNameAndAddress(User user);
}
```
- `type`：sql语句所在类的字节码
- `method`：返回sql语句的方法

更多动态sql语法请见[官方文档](https://mybatis.org/mybatis-3/zh/statement-builders.html)
