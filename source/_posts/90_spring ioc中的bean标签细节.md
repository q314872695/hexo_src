---
title: spring IOC中的bean标签细节
tags: spring
abbrlink: 77525db8
date: 2020-06-07 22:03:28
---

# bean标签
- 作用：
	- 用于配置让spring来创建的对象。
	- 默认情况下它调用的是类中的无参构造函数。如果没有无参构造函数则不能创建成功。
- 属性：
	- id：给对象在容器中提供一个唯一标识。用于获取对象。
	- class：指定类的全限定类名。用于反射创建对象。默认情况下调用无参构造函数。
	- scope：指定对象的作用范围。
		- `singleton`：默认值，单例的.
		- `prototype`：多例的.
		- `request`：WEB 项目中,Spring 创建一个 Bean 的对象,将对象存入到`request`域中.
		- `session`：WEB 项目中,Spring 创建一个 Bean 的对象,将对象存入到`session`域中.
		- `application`：WEB 项目中,Spring 创建一个 Bean 的对象,将对象存入到`ServletContext`域中.
		- `websocket`：将单个bean定义的作用域限定为的生命周期`WebSocket`。
	- init-method：指定类中的初始化方法名称。
	- destroy-method：指定类中销毁方法名称。
# bean 的作用范围和生命周期
## 单例对象：`scope="singleton"`
一个应用只有一个对象的实例。它的作用范围就是整个引用。

生命周期：
- 对象出生：当应用加载，创建容器时，对象就被创建了。
- 对象活着：只要容器在，对象一直活着。
- 对象死亡：当应用卸载，销毁容器时，对象就被销毁了。

## 多例对象：`scope="prototype"`
每次访问对象时，都会重新创建对象实例。

生命周期：
- 对象出生：当使用对象时，创建新的对象实例。
- 对象活着：只要对象在使用中，就一直活着。
- 对象死亡：当对象长时间不用时，被 java 的垃圾回收器回收了。

# 实例化 Bean 的三种方式
## 使用默认无参构造函数
```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"/>
```
## spring 管理静态工厂-使用静态工厂的方法创建对象
java类
```java
/**
* 模拟一个静态工厂，创建业务层实现类
*/
public class StaticFactory {
	public static IAccountService createAccountService(){
		return new AccountServiceImpl();
	} 
}
```
spirng配置文件
```xml
<bean id="accountService" class="com.itheima.factory.StaticFactory" factory-method="createAccountService"></bean>
```
- id 属性：指定 bean 的 id，用于从容器中获取
- class 属性：指定静态工厂的全限定类名
- factory-method 属性：指定生产对象的静态方法
## spring 管理实例工厂-使用实例工厂的方法创建对象
```java
/**
* 模拟一个实例工厂，创建业务层实现类
* 此工厂创建对象，必须现有工厂实例对象，再调用方法
*/
public class InstanceFactory {
	public IAccountService createAccountService(){
		return new AccountServiceImpl();
	} 
}
```
```xml
<bean id="instancFactory" class="com.itheima.factory.InstanceFactory"></bean>
<bean id="accountService" factory-bean="instancFactory" factory-method="createAccountService"></bean>
```
- factory-bean 属性：用于指定实例工厂 bean 的 id。
- factory-method 属性：用于指定实例工厂中创建对象的方法。
# Spring的依赖注入
## 构造函数注入
```java
public class Person {
    private String name;
    private int age;
    private Date birthday;

    public Person() {
    }

    public Person(String name, int age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                '}';
    }
}

```
**涉及的标签：**
- `constructor-arg`:
	- 属性：
		- `index`:指定参数在构造函数参数列表的索引位置
		- `type`:指定参数在构造函数中的数据类型
		- `name`:指定参数在构造函数中的名称 用这个找给谁赋值（常用）
		- `value`:它能赋的值是基本数据类型和 String 类型
		- `ref`:它能赋的值是其他 bean 类型，也就是说，必须得是在配置文件中配置过的 bean

**修改配置文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="person" class="com.itheima.Person">
        <constructor-arg name="name" value="张三"/>
        <constructor-arg name="age" value="18"/>
        <constructor-arg name="birthday" ref="date"/>
    </bean>
    <bean id="date" class="java.util.Date"/>
</beans>
```
##  set 方法注入
**修改配置文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="person" class="com.itheima.Person">
        <property name="name" value="张三"/>
        <property name="age" value="20"/>
        <property name="birthday" ref="date"/>
    </bean>
    <bean id="date" class="java.util.Date"/>
</beans>
```
## 注入集合属性
顾名思义，就是给类中的集合成员传值，它用的也是set方法注入的方式，只不过变量的数据类型都是集合。
这里介绍注入数组，List,Set,Map,Properties。具体代码如下：
```java
public class A {
    private String[] myStrs;
    private List<String> myList;
    private Set<String> mySet;
    private Map<String,String> myMap;
    private Properties myProps;

    public String[] getMyStrs() {
        return myStrs;
    }

    public void setMyStrs(String[] myStrs) {
        this.myStrs = myStrs;
    }

    public List<String> getMyList() {
        return myList;
    }

    public void setMyList(List<String> myList) {
        this.myList = myList;
    }

    public Set<String> getMySet() {
        return mySet;
    }

    public void setMySet(Set<String> mySet) {
        this.mySet = mySet;
    }

    public Map<String, String> getMyMap() {
        return myMap;
    }

    public void setMyMap(Map<String, String> myMap) {
        this.myMap = myMap;
    }

    public Properties getMyProps() {
        return myProps;
    }

    public void setMyProps(Properties myProps) {
        this.myProps = myProps;
    }
}
```
List 结构的：
- array
- list
- set

Map 结构的
- map
- Properties

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="a" class="com.itheima.A">
        <!-- 在注入集合数据时，只要结构相同，标签可以互换 -->
        <!-- 给数组注入数据 -->
        <property name="myStrs">
            <array>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </array>
        </property>
        <!-- 注入 list 集合数据 -->
        <property name="myList">
            <list>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </list>
        </property>
        <!-- 注入 set 集合数据 -->
        <property name="mySet">
            <set>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </set>
        </property>
        <!-- 注入 Map 数据 -->
        <property name="myMap">
            <map>
                <entry key="testA" value="aaa"></entry>
                <entry key="testB">
                    <value>bbb</value>
                </entry>
            </map>
        </property>
        <!-- 注入 properties 数据 -->
        <property name="myProps">
            <props>
                <prop key="testA">aaa</prop>
                <prop key="testB">bbb</prop>
            </props>

        </property>
    </bean>

</beans>
```