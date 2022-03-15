---
title: mybatis快速入门
tags:
  - java
  - mybatis
abbrlink: ba767012
date: 2020-05-31 21:48:18
---

# 搭建mybatis开发环境
## 基于xml
1. 创建Maven工程，在`pom.xml`文件中添加一下坐标
```xml
<dependencies>
        <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.4</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.26</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/log4j/log4j -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
    </dependencies>
```
2. 编写User实体类
每个实体类对应数据库中得一张表，实体类的成员变量名应尽量和数据库表的列名一致，这样可以减少一部分操作。

```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", birthday=" + birthday +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```
3. 编写持久层接口`IUserDao`
```java
public interface IUserDao {
    /**
     * 查询所有用户
     * @return
     */
    List<User> findAll();
}
```
4. 编写持久层接口的映射文件`IUserDao.xml`

**要求：**
- 创建位置：必须和持久层接口在相同的包中。
- 名称：必须是以持久层接口命名的文件名，扩展名是xml

如图：
![image.png](https://halo-1257208482.image.myqcloud.com/image_1590931348381.png!webp)

`resources`中的文件夹只能一个一个单独创建，否则可能是这样（其实是创建了一个文件夹）：
![image.png](https://halo-1257208482.image.myqcloud.com/image_1590931448209.png!webp)

**IUserDao.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.IUserDao">
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user
    </select>
</mapper>

namespace：告诉mybatis该映射文件与哪个接口对应
```

5. 编写`SqlMapConfig.xml`配置文件，存放于
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--配置环境-->
    <environments default="mysql">
<!--配置mysql环境-->
        <environment id="mysql">
<!--配置事务类型-->
            <transactionManager type="JDBC"/>
<!--配置数据源（连接池）-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
<!--告诉mybatis映射文件的位置-->
    <mappers>
        <mapper resource="com/itheima/dao/IUserDao.xml"/>
    </mappers>
</configuration>
```
6. 编写日志配置文件`log4j.properties`(可选，可以查看mybatis运行过程中的相关信息)
```properties
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=debug, CONSOLE,LOGFILE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%t] %-5p %30.30c %x - %m%n

# LOGFILE is set to be a File appender using a PatternLayout.
log4j.appender.LOGFILE=org.apache.log4j.FileAppender
log4j.appender.LOGFILE.File=d:\\axis.log
log4j.appender.LOGFILE.Append=true
log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r [%t] %-5p %30.30c %x - %m%n
```
 
7. 编写测试类
```java
public class MybatisTest {
    public static void main(String[] args) throws IOException {
        // 读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 使用构建者创建工厂对象
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        // 使用工厂对象生产sqlsession对象
        SqlSession session = factory.openSession();
        // 使用sqlsession创建IUserDao接口的代理对象
        IUserDao userDao = session.getMapper(IUserDao.class);
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }
        session.close();
        in.close();
    }
}
```
## 基于注解
1. 修改持久层IUserDao接口中添加注解
```java
public interface IUserDao {
    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from user")
    List<User> findAll();
}
```
2. 修改`SqlMapConfig.xml`文件
```xml
<!--告诉mybatis映射文件的位置-->
<mappers>
	<mapper class="com.itheima.dao.IUserDao"/>
</mappers>
```
3. 删除`IUserDao.xml`文件，否则会报错


