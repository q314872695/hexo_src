---
title: spring中的AOP
tags: spring
abbrlink: affc4263
date: 2020-06-09 10:43:23
---

# AOP的作用和优势
**作用：**

在程序运行期间，不修改源码对已有方法进行增强。

**优势：**
- 减少重复代码
- 提高开发效率
- 维护方便
# 基于xml的AOP配置
## 准备用于测试的代码
用于模拟打印日志的类，也叫做切面类
```java
@Component
public class PrintLog {
    /**
     * 前置通知
     */
    public void beforeLog() {
        System.out.println("方法执行前打印了一条日志");
    }

    /**
     * 后置通知
     */
    public void afterReturningLog() {
        System.out.println("方法执行后打印了一条日志");
    }

    /**
     * 异常通知
     */
    public void afterThrowingLog() {
        System.out.println("方法出现异常时打印了一条日志");
    }

    /**
     * 最终通知
     */
    public void afterLog() {
        System.out.println("方法最后打印了一条日志，无论是否报异常都会打印");
    }

    /**
     * 环绕通知
     */
    public Object roundLog(ProceedingJoinPoint proceedingJoinPoint) {
        Object object = null;
        try {
            // 调用前置
            beforeLog();
            // 获取被增强方法的参数
            Object[] args = proceedingJoinPoint.getArgs();
            // 执行被增强的方法
            object = proceedingJoinPoint.proceed(args);
            // 后置前置
            afterReturningLog();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            // 异常前置
            afterThrowingLog();
        } finally {
            // 最终前置
            afterLog();
        }
        return object;
    }
}
```
执行该方法时打印日志，也叫做通知类
```java
@Service("accountService")
public class AccountServiceImpl implements IAccountService {

    public void findAccountAll() {
        System.out.println("查询账户的方法执行了");
    }
}
```
## 在pom.xml文件导入相关坐标
```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.6.RELEASE</version>
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
        <!--用于解析切入点表达式-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.5</version>
        </dependency>
```
## 在spring配置文件中对 aop 配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--告诉spring在创建容器时，要扫描的包-->
    <context:component-scan base-package="com.itheima"/>

    <!--用于声明开始 aop 的配置-->
    <aop:config>
        <aop:pointcut id="pt" expression="execution(* com.itheima.server.impl.*.*(..))"/>
        <!--引用了用于打印方法的类-->
        <aop:aspect id="txAdvice" ref="printLog">
            <!--配置前置通知，在方法之前执行-->
            <aop:before method="beforeLog" pointcut-ref="pt"/>
            <!--配置后置通知，在方法之后执行，切点表达式也可配置在通知中-->
            <aop:after-returning method="afterReturningLog" pointcut="execution(* com.itheima.server.impl.*.*(..))"/>
            <!--配置异常通知，在方法产生异常时执行-->
            <aop:after-throwing method="afterThrowingLog" pointcut="execution(* com.itheima.server.impl.*.*(..))"/>
            <!--配置最终通知，在方法是否发生异常都会执行执行-->
            <aop:after method="afterLog" pointcut="execution(* com.itheima.server.impl.*.*(..))"/>
            <!--配置环绕通知,手动控制增强代码什么时候执行，可以代替上面的4个通知类型-->
            <!--<aop:around method="roundLog" pointcut-ref="pt"/>-->
        </aop:aspect>
    </aop:config>
</beans>
```
## 测试
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:ApplicationContext.xml")
public class AccountSereviceTest {
    @Autowired
    private IAccountService accountService;

    @Test
    public void test1(){
        accountService.findAccountAll();

    }
}
结果：
方法执行前打印了一条日志
查询账户的方法执行了
方法执行后打印了一条日志
方法最后打印了一条日志，无论是否报异常都会打印
```
**相关知识点：**
- `aop:config`：用于声明开始 aop 的配置
- `aop:aspect`：用于配置切面。
	- 属性：
		- id：给切面提供一个唯一标识。
		- ref：引用配置好的通知类 bean 的 id
- `aop:pointcut`：用于配置切入点表达式，就是指定对哪些类的哪些方法进行增强。
	- expression：用于定义切入点表达式。
	- id：用于给切入点表达式提供一个唯一标识
- `aop:before`：用于配置前置通知。指定增强的方法在切入点方法之前执行
	- 属性：
		- method:用于指定通知类中的增强方法名称
		- ponitcut-ref：用于指定切入点的表达式的引用
		- poinitcut：用于指定切入点表达式
- `aop:after-returning`：用于配置后置通知
	- 属性：同`aop:before`
- `aop:after-throwing`：用于配置异常通知
	- 属性：同`aop:before`
- `aop:after`：用于配置最终通知
	- 属性：同`aop:before`
- `aop:around`：用于配置环绕通知
	- 属性：同`aop:before`
	- 说明：它是 spring 框架为我们提供的一种可以在代码中手动控制增强代码什么时候执行的方式，通常情况下，环绕通知都是独立使用的，它可以代替前面的4个通知。
# 基于注解的AOP配置
## 在打印日志的类中加添相关注解
```java
package com.itheima.utils;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Component
@Aspect   //表明当前类是一个切面类
public class PrintLog {
    @Pointcut("execution(* com.itheima.service.impl.*.*(..))")
    private void pt(){}
    /**
     * 前置通知
     */
    @Before("execution(* com.itheima.service.impl.*.*(..))")
    public void beforeLog() {
        System.out.println("方法执行前打印了一条日志");
    }

    /**
     * 后置通知
     */
    @AfterReturning("execution(* com.itheima.service.impl.*.*(..))")
    public void afterReturningLog() {
        System.out.println("方法执行后打印了一条日志");
    }

    /**
     * 异常通知
     */
    @AfterThrowing("execution(* com.itheima.service.impl.*.*(..))")
    public void afterThrowingLog() {
        System.out.println("方法出现异常时打印了一条日志");
    }

    /**
     * 最终通知
     */
    @After("pt()") //注意：千万别忘了写括号
    public void afterLog() {
        System.out.println("方法最后打印了一条日志，无论是否报异常都会打印");
    }

    /**
     * 环绕通知
     */
//    @Around("pt()")
    public Object roundLog(ProceedingJoinPoint proceedingJoinPoint) {
        Object object = null;
        try {
            // 调用前置
            beforeLog();
            // 获取被增强方法的参数
            Object[] args = proceedingJoinPoint.getArgs();
            // 执行被增强的方法
            object = proceedingJoinPoint.proceed(args);
            // 后置前置
            afterReturningLog();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            // 异常前置
            afterThrowingLog();
        } finally {
            // 最终前置
            afterLog();
        }
        return object;
    }
}
```
**相关知识：**
- `@Aspect`：表明当前类是一个切面类
- `@Before`：把当前方法看成是前置通知。
	- 属性：
		- value：用于指定切入点表达式，还可以指定切入点表达式的引用。
- `@AfterReturning`：把当前方法看成是后置通知。
	- 属性：同上
- `@AfterThrowing`：把当前方法看成是异常通知。
	- 属性：同上
- `@After`：把当前方法看成是最终通知。
	- 属性：同上
- `@Around`：把当前方法看成是环绕通知。
	- 属性：同上
- `@Pointcut`：指定切入点表达式

## 修改配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--告诉spring在创建容器时，要扫描的包-->
    <context:component-scan base-package="com.itheima"/>
    <!--开启注解版的AOP功能-->
    <aop:aspectj-autoproxy/>
</beans>
```
## 测试
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:ApplicationContext.xml")
public class AccountServiceTest {
    @Autowired
    private AccountServiceImpl accountService;

    @Test
    public void tset1(){
        accountService.findAccountAll();
    }
}
结果：
方法执行前打印了一条日志
查询账户的方法执行了
方法最后打印了一条日志，无论是否报异常都会打印
方法执行后打印了一条日志
```
**我们发现有一个bug，最终通知在后置通知之前，而正确的执行顺序时：前置通知、后置通知（异常通知）、最终通知**

**解决办法：**
可以通过使用环绕通知来代替上述四个通知。















