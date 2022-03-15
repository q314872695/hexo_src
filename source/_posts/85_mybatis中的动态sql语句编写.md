---
title: mybatis中的动态sql语句编写
tags:
  - java
  - mybatis
abbrlink: 90c0f7e
date: 2020-06-04 14:43:25
---

# 动态 SQL 之`<if>`标签
>我们根据实体类的不同取值，使用不同的 SQL 语句来进行查询。比如在 id 如果不为空时可以根据 id 查询，如果 username 不同空时还要加入用户名作为条件。这种情况在我们的多条件组合查询中经常会碰到。

**持久层 Dao 接口**
```java
/**
* 根据用户信息，查询用户列表
* @param user
* @return
*/
List<User> findUserByCondition(User user);
```
**持久层 Dao 映射配置**
```xml
<select id="findUserByCondition" parameterType="com.itheima.domain.User" resultType="com.itheima.domain.User">
        select * from user where 1=1
        <if test="username!=null">
            and username=#{username}
        </if>
        <if test="sex!=null">
            and sex=#{sex}
        </if>
</select>
```
> where 1=1 为了方便添加后面的sql条件，`<if>`标签的 test 属性中写的是对象的属性名

**测试**
```java
@Test
public void testFindByCondition(){
	User u = new User();
        u.setUsername("老王");
        u.setSex("女");
        List<User> users = userDao.findUserByCondition(u);
        for (User user : users) {
		System.out.println(user);
        }
}
```
# 动态 SQL 之`<where>`标签
> 为了简化上面 where 1=1 的条件拼装，我们可以采用<where>标签来简化开发。

**修改上面案例中的持久层 Dao 映射配置**
```xml
<select id="findUserByCondition" parameterType="com.itheima.domain.User" resultType="com.itheima.domain.User">
        select * from user
        <where>
            <if test="username!=null">
                and username=#{username}
            </if>
            <if test="sex!=null">
                and sex=#{sex}
            </if>
        </where>
</select>
```
# 动态标签之`<foreach>`标签
> 实现传入多个id 查询用户信息，sql：select * from user where id in (41,41,46)

**在 QueryVo 中加入一个 List 集合用于封装参数**
```java
public class QueryVo {

    private List<Integer> ids;

    public List<Integer> getIds() {
        return ids;
    }

    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }
}
```
**持久层 Dao 接口**
```java
/**
* 根据 id 集合查询用户
* @param vo
* @return
*/
List<User> findInIds(QueryVo vo);
```
**持久层 Dao 映射配置**
```xml
<!--    select * from user where id in (41,41,46)-->
<select id="findUserByInIds" resultType="com.itheima.domain.User" parameterType="com.itheima.domain.QueryVo">
	select * from user
        <where>
            <if test="ids!=null and ids.size()>0">
                <foreach collection="ids" open="id in(" close=")" item="uid" separator=",">
                    #{uid}
                </foreach>
            </if>
        </where>
</select>
```
- `<foreach>`标签用于遍历集合，它的属性：
	- `collection`:代表要遍历的集合元素，注意编写时不要写`#{}`
	- `open`:代表语句的开始部分
	- `close`:代表结束部分
	- `item`:代表遍历集合的每个元素，生成的变量名(和`foreach`中的`#{}`个名称一致即可)
	- `sperator`:代表分隔符

**测试**
```java
@Test
public void testFindByInIds(){
        QueryVo vo = new QueryVo();
        vo.setIds(Arrays.asList(41,42,46));
        List<User> users = userDao.findUserByInIds(vo);
        for (User user : users) {
            System.out.println(user);
        }
}
```