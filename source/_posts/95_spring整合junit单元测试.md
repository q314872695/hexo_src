---
title: spring整合junit单元测试
tags:
  - java
  - spring
abbrlink: 4045f8ae
date: 2020-06-08 21:27:43
---

# 前情提要
在之前的测试代码中都有一下两行代码：
```java
applicationContext = new AnnotationConfigApplicationContext(SpringConfig.class);
accountService = applicationContext.getBean("accountService", IAccountService.class);
```
比较麻烦。可以在测试的方法中直接从spring容器中获取对象。
# 步骤
## 导入相关jar包
```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<version>5.2.6.RELEASE</version>
	<scope>test</scope>
</dependency>
```
## 使用@RunWith 注解替换原有运行器
## 使用@ContextConfiguration 指定 spring 配置文件的位置
- `@ContextConfiguration`注解：
	- `locations`属性：用于指定配置文件的位置。如果是类路径下，需要用 classpath:表明
	- `classes`属性：用于指定注解的类。当不使用 xml 配置时，需要用此属性指定注解类的位置
## 使用@Autowired 给测试类中的变量注入数据

```xml
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {
    @Autowired
    private IAccountService accountService;



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
。