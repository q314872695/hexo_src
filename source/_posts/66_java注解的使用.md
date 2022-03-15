---
title: java注解的使用
tags: java
abbrlink: aad189ba
date: 2020-05-17 09:47:11
---

# 什么是注解
注解（Annotation），也叫元数据。一种代码级别的说明。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。

**和注释的区别**
- 注释：用文字描述程序的。给程序员看的
- 注解：说明程序的。给计算机看的

**作用分类**
- 编写文档：通过代码里标识的注解生成文档【生成文档doc文档】
- 代码分析：通过代码里标识的注解对代码进行分析【使用反射】
- 编译检查：通过代码里标识的注解让编译器能够实现基本的编译检查【Override】

**JDK中预定义的一些注解**
- `@Override`：检测被该注解标注的方法是否是继承自父类(接口)
- `@Deprecated`：该注解标注的内容，表示已过时
- `@SuppressWarnings`：压制警告，一般传递参数all,`@SuppressWarnings("all")`

# 自定义注解

**格式**
```java
元注解
public @interface 注解名称{
	属性列表;
}
```

**元注解**

元注解就是用于描述注解的注解，常用的元注解如下：
- `@Target`：描述注解能够作用的位置（不写则哪个位置都可以）
	- 类或接口：`ElementType.TYPE`
	- 字段：`ElementType.FIELD`
	- 方法：`ElementType.METHOD`
	- 构造方法：`ElementType.CONSTRUCTOR`
	- 方法参数：`ElementType.PARAMETER`
- `@Retention`：描述注解被保留的阶段
	- 仅编译期：`RetentionPolicy.SOURCE`，例如`@Override`，该注解在编译期就被丢掉了
	- 仅class文件：`RetentionPolicy.CLASS`(默认)，该注解仅保存在class文件中，它们不会被加载进JVM
	- 运行期：`RetentionPolicy.RUNTIME`(最最最常用)，该注解会被加载进JVM，并且在运行期可以被程序读取



例如定义注解`@MyAnno`可用在方法上：
```java
@Target(ElementType.METHOD)
public @interface MyAnno{
 	// 注解中的属性
}

// 注解的本质就是一个接口，该接口默认继承Annotation接口,根据反编译注解的class文件得到
public interface MyAnno extends java.lang.annotation.Annotation {
	// ...
}
```

**属性**

注解中的属性就相当于接口中的抽象方法，对该“抽象方法”的返回值有如下要求：
- 基本数据类型
- String
- 枚举
- 注解
- 以上类型的数组

**关于属性的注意事项：**
1. 如果定义属性时，使用default关键字给属性默认初始化值，则使用注解时，可以不进行属性的赋值。
2. 如果只有一个属性需要赋值，并且属性的名称是value，则value可以省略，直接定义值即可。
3. 数组赋值时，值使用{}包裹。如果数组中只有一个值，则{}可以省略

# 使用（解析）注解
`@Target(RetentionPolicy.RUNTIME)`修饰的注意最最常用，所以仅对它修饰的注解进行解析。

**解析：获取注解中定义的属性值**

1. 获取注解定义的位置的对象  （Class，Method，Field，Constructor）
2. 判断某个注解是否存在于Class、Field、Method或Constructor：
	- `Class.isAnnotationPresent(Class annotationClass)` 返回值均是boolean类型
	- `Field.isAnnotationPresent(Class annotationClass)`
	- `Method.isAnnotationPresent(Class annotationClass)`
	- `Constructor.isAnnotationPresent(Class annotationClass)`
3. 获取指定的注解`getAnnotation(Class annotationClass)`
4. 调用注解中的抽象方法获取配置的属性值

**例如：**
```java
// 先定义个注解
@Retention(RetentionPolicy.RUNTIME)
public @interface Report {
    String value() default "";
}
// 再定义个类
@Report("哇哈哈")
public class Person {
    @Report("张三")
    String name;
}


public class Main {
    public static void main(String[] args) throws NoSuchFieldException {
        // 解析类的注解
        Class<Person> personClass = Person.class;
	// 判断该类有注解就进行解析
        if (personClass.isAnnotationPresent(Report.class)) {
            Report annotation = personClass.getAnnotation(Report.class);
            String value = annotation.value();
            System.out.println(value);
        }
        // 解析类中属性的注解
        Field field = personClass.getDeclaredField("name");
        if (field.isAnnotationPresent(Report.class)) {
            Report annotation1 = field.getAnnotation(Report.class);
            String value1 = annotation1.value();
            System.out.println(value1);
        }
        
    }
}
```




