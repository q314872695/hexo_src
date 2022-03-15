---
title: 使用spring 注解+xml配置方式对数据库进行CRUD操作
tags:
  - java
  - spring
abbrlink: 768421c6
date: 2020-06-08 16:58:07
---

> 注解加xml配置时比较常用的方式，对于自己写的类使用注解，对于别人写的类（如使用jar包）就在xml中配置

# 环境搭建
## 使用到的技术
 - spring：spring容器
 - jdbc template：spring对JDBC的简单封装
 - c3p0：数据库连接池
 - lombok：简化javabean的代码

## 在pom.xml中导入坐标
```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.5</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
```
## 编写实体类
```java
@Data
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Float money;
}
```
## 编写持久层接口以及实现类
```java
public interface IAccountDao {
    List<Account> findAccountAll();

    Account findAccountById(Integer id);

    int saveAccount(Account account);

    int deleteAccount(Integer id);

    int updateAccount(Account account);
}
```
```java
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {
    @Autowired
    private JdbcTemplate template;

    public List<Account> findAccountAll() {

        return template.query("select * from account", new BeanPropertyRowMapper<Account>(Account.class));
    }

    public Account findAccountById(Integer id) {
        return template.queryForObject("select * from where id=?", new BeanPropertyRowMapper<Account>(Account.class), id);
    }

    public int saveAccount(Account account) {
        return template.update("insert into account(name,money) values(?,?)", account.getName(), account.getMoney());
    }

    public int deleteAccount(Integer id) {
        return template.update("delete from account where id=?", id);
    }

    public int updateAccount(Account account) {
        return template.update("update account set name=?,money=? where id=?", account.getName(), account.getMoney(), account.getId());
    }
}
```
## 编写业务层接口及其实现类
```java
public interface IAccountService {
    List<Account> findAccountAll();

    Account findAccountById(Integer id);

    int saveAccount(Account account);

    int deleteAccount(Integer id);

    int updateAccount(Account account);
}
```
```java
/*
* 该案例较为简单，所以业务层直接调用持久层的方法
* */
@Service("accountService")
public class AccountServiceImpl implements IAccountService {
    @Autowired
    private IAccountDao accountDao;

    public List<Account> findAccountAll() {
        return accountDao.findAccountAll();
    }

    public Account findAccountById(Integer id) {
        return accountDao.findAccountById(id);
    }

    public int saveAccount(Account account) {
        return accountDao.saveAccount(account);
    }

    public int deleteAccount(Integer id) {
        return accountDao.deleteAccount(id);
    }

    public int updateAccount(Account account) {
        return accountDao.updateAccount(account);
    }
}
```
## 编写spring配置文件ApplicationContext.xml
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
    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/spring"/>
        <property name="user" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <!--将jdbc template存入spring容器-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate" scope="prototype">
        <constructor-arg name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```
# 测试
```java
public class AccountServiceTest {
    private ApplicationContext applicationContext;
    private IAccountService accountService;

    @Before
    public void init() {
        applicationContext = new ClassPathXmlApplicationContext("ApplicationContext.xml");
        accountService = applicationContext.getBean("accountService", IAccountService.class);
    }

    @Test
    public void test1() {
        List<Account> accounts = accountService.findAccountAll();
        for (Account account : accounts) {
            System.out.println(account);
        }
    }

    @Test
    public void test2() {
        Account account = accountService.findAccountById(2);
        System.out.println(account);

    }

    @Test
    public void test3() {
        Account account = new Account();
        account.setId(4);
        account.setName("ddd");
        account.setMoney(1500f);
        accountService.saveAccount(account);

    }

}
```
