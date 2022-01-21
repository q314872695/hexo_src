---
title: spring中的声明式事务控制
date: 2020-06-09 18:21:55
tags:
- java
- spring
---

# 事务控制的API介绍
- `PlatformTransactionManager`：此接口是 spring 的事务管理器，它里面提供了我们常用的操作事务的方法。
	- `TransactionStatus getTransaction(TransactionDefinition var1)`：获取事务状态信息
	- `void commit(TransactionStatus var1)`：提交事务
	- `void rollback(TransactionStatus var1)`：回滚事务
	- 常用实现类：`DataSourceTransactionManager`使用 Spring JDBC 或 iBatis 进行持久化数据时使用
- `TransactionDefinition`：它是事务的定义信息对象，里面有如下方法：
	- `String getName()`：获取事务对象名称
	- `int getIsolationLevel()`：获取事务隔离级别
		- `ISOLATION_READ_UNCOMMITTED`：可以读取未提交的数据，产生的问题：脏读、不可重复读、幻读
		- `ISOLATION_READ_COMMITTED`：只能读取已提交的数据（Oracle默认），产生的问题：不可重复读、幻读
		- `ISOLATION_REPEATABLE_READ`：可重复读（MySQL默认），产生的问题：幻读
		- `ISOLATION_SERIALIZABLE`：串行化，可以解决所有的问题
	- `int getPropagationBehavior()`：获取事务传播行为
		- `REQUIRED`:如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。一般的选
择（默认值）
		- `SUPPORTS`:支持当前事务，如果当前没有事务，就以非事务方式执行（没有事务）
	- `int getTimeout()`：获取事务超时时间
		- 默认值是-1，没有超时限制。如果有，以秒为单位进行设置。
	- `boolean isReadOnly()`：获取事务是否可读
		- 建议查询时设置为只读。
- `TransactionStatus`：此接口提供的是事务具体的运行状态。
	- `void flush()`：刷新事务
	- `boolean hasSavepoint()`：获取是否存在存储点
	- `boolean isCompleted()`：获取事务是否已完成
	- `boolean isNewTransaction()`：获取事务是否为新的事务
	- `boolean isRollbackOnly()`：获取事务是否回滚
	- `void setRollbackOnly()`：设置事务回滚
# 基于xml事务控制
## 需求
模拟两个账户转账，保证事务一致性
## 导入pom.xml坐标
```xml
<dependencies>
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
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.2.6.RELEASE</version>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
        <!--用于解析切入点表达式-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.5</version>
        </dependency>


    </dependencies>
```
## 创建 spring 的配置文件并导入约束
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx https://www.springframework.org/schema/tx/spring-tx.xsd">


</beans>
```
## 代码准备
持久层接口
```java
public interface IAccountDao {
    Account findAccountByName(String name);
    int updateAccount(Account account);
}
```
持久层实现类
```java
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {
    @Autowired
    private JdbcTemplate template;

    public Account findAccountByName(String name) {
        return template.queryForObject("select * from account where name=?", new BeanPropertyRowMapper<Account>(Account.class), name);
    }

    public int updateAccount(Account account) {
        return template.update("update account set name=?,money=? where id=?", account.getName(), account.getMoney(), account.getId());
    }

}
```
业务层接口
```java
public interface IAccountService {
    // 模拟转账的接口
    void transfer(String sourceName, String targetName, Float money);
}
```
业务层实现类
```java
@Service("accountService")
public class AccountServiceImpl implements IAccountService {
    @Autowired
    private IAccountDao accountDao;

    public void transfer(String sourceName, String targetName, Float money) {
        Account source = accountDao.findAccountByName(sourceName);
        Account target = accountDao.findAccountByName(targetName);
        source.setMoney(source.getMoney() - money);
        target.setMoney(target.getMoney() + money);

        accountDao.updateAccount(source);
        int i = 1 / 0;  // 用于产生异常测试事务一致性
        accountDao.updateAccount(target);
    }
}
```
## 配置业务层，持久层
```java
    <!--告诉spring在创建容器时，要扫描的包-->
    <context:component-scan base-package="com.itheima"/>
    <!--配置数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring?useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <!--将jdbc template存入spring容器-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate" scope="prototype">
        <constructor-arg name="dataSource" ref="dataSource"/>
    </bean>
```

## 事务控制步骤
1. 配置事务管理器
2. 配置事务的通知，并且引用事务管理器
3. 配置事务的属性
4. 配置 AOP 切入点表达式
5. 配置切入点表达式和事务通知的对应关系

在配置文件中添加以下内容：
```xml
    <!--声明式事务控制配置-->
    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置事务的通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!--配置事务的属性-->
        <!-- 指定方法名称：是业务核心方法
        read-only：是否是只读事务。默认 false，不只读。
        isolation：指定事务的隔离级别。默认值是使用数据库的默认隔离级别。
        propagation：指定事务的传播行为。
        timeout：指定超时时间。默认值为：-1。永不超时。
        rollback-for：用于指定一个异常，当执行产生该异常时，事务回滚。产生其他异常，事务不回滚。
        没有默认值，任何异常都回滚。
        no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时，事务回滚。没有默认值，任何异常都回滚。-->
        <tx:attributes>
            <!--查询方法不开启事务-->
            <tx:method name="find*"  propagation="SUPPORTS" read-only="true"/>
            <!--除了find开头的方法，都开启事务-->
            <tx:method name="*"  propagation="REQUIRED" read-only="false"/>
        </tx:attributes>
    </tx:advice>
    <!--配置AOP-->
    <aop:config>
<!--        配置切入点表达式-->
        <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"/>
<!--        建立切入点表达式和事务通知的对应关系-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
    </aop:config>
```
## 测试结果
转账方法出现异常时，数据原封不动；没有异常时正常执行

# 基于注解的事务控制
1. 配置事务管理器并注入数据源
2. 在业务层使用`@Transactional`注解
	- `@Transactional(readOnly=false,propagation=Propagation.REQUIRED)`：这是默认的，不写也可以，适用于需要事务控制的方法
	- `@Transactional(readOnly=true,propagation=Propagation.SUPPORTS)`：此方式没有事务控制，可以在查询的方法上单独配置注解
```java
@Service("accountService")
@Transactional
public class AccountServiceImpl implements IAccountService {
    @Autowired
    private IAccountDao accountDao;

    public void transfer(String sourceName, String targetName, Float money) {
        Account source = accountDao.findAccountByName(sourceName);
        Account target = accountDao.findAccountByName(targetName);
        source.setMoney(source.getMoney() - money);
        target.setMoney(target.getMoney() + money);

        accountDao.updateAccount(source);
        int i = 1 / 0;
        accountDao.updateAccount(target);
    }
}
```
3. 在配置文件中开启 spring 对注解事务的支持

**配置文件：**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx https://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--告诉spring在创建容器时，要扫描的包-->
    <context:component-scan base-package="com.itheima"/>
    <!--spring内置数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring?useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <!--将jdbc template存入spring容器-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate" scope="prototype">
        <constructor-arg name="dataSource" ref="dataSource"/>
    </bean>
    <!--声明式事务控制配置-->
    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--开启spring对注解事务的支持-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```
