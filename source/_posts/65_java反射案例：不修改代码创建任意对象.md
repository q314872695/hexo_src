---
title: java反射案例：不修改代码创建任意对象
tags: java
abbrlink: 120f2392
date: 2020-05-17 09:32:01
---

# 需求
写一个“框架”，不能改变该类的任何代码的前提下，可以帮我们创建任意类的对象，并且执行其中任意方法。

# 步骤
1. 将需要创建的对象的全类名和需要执行的方法定义在配置文件中
2. 在程序中加载读取配置文件
3. 使用反射技术来加载类文件进内存
4. 创建对象
5. 执行方法

# 实现
Student.java
```java
// 先写个类，一会要创建它
package reflect;

public class Student {
    public void sleep(){
        System.out.println("sleep...");
    }
}
```
再src目录下创建个配置文件`pro.properties`
```java
# 要创建的类，写全类名
className=reflect.Student
# 要运行的方法
methodName=sleep
```
写主要的代码
```java
public class Demo4 {
    public static void main(String[] args) throws Exception {
	// 加载配置文件
        Properties properties = new Properties();
        ClassLoader classLoader = Demo4.class.getClassLoader();
        InputStream is = classLoader.getResourceAsStream(".\\pro.properties");
        properties.load(is);

	// 获取配置文件中定义的数据
        String className = properties.getProperty("className");
        String methodName = properties.getProperty("methodName");


	// 加载该类进内存
        Class cls = Class.forName(className);
	// 创建对象
        Object obj = cls.newInstance();
	// 创建方法对象
        Method method = cls.getMethod(methodName);
	// 执行该方法
        method.invoke(obj);
    }
}

运行结果：
sleep...
```