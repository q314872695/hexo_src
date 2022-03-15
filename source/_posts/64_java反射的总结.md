---
title: java反射的总结
tags: java
abbrlink: 18319d91
date: 2020-05-16 22:24:58
---

java框架中绝大部分都运用了反射技术，所谓框架就是半成品的软件，在框架的基础上进行软件开发可以简化代码。**反射是为了解决在运行期，对某个实例一无所知的情况下，如何调用其方法**

Java的反射是指程序在运行期可以拿到一个对象的所有信息。
# 获取Class对象的方式
首先先了解一下java代码的三个阶段
![image.png](https://halo-1257208482.image.myqcloud.com/image_1589680111300.png!webp)

**以下3中方式分别为不同阶段获取class对象**
1. `Class.forName("全类名")`：将字节码文件加载进内存，返回Class对象。多用于配置文件，将类名定义在配置文件中。读取文件，加载类。
2. `类名.class`：通过类名的属性class获取。多用于参数的传递。
3. `对象名称.getClass()`：getClass()方法在Object类中定义着。多用于对象的获取字节码的方式。

同一个字节码文件(*.class)在一次程序运行过程中，只会被加载一次，不论通过哪一种方式获取的Class对象都是同一个。

# Class对象功能
## 获取成员变量们
- `Field getField(String name)`：根据字段名获取某个public的field（包括父类）
- `Field[] getFields()` ：获取所有public的field（包括父类）
- `Field getDeclaredField(String name)`：根据字段名获取当前类的某个field（不包括父类）
- `Field[] getDeclaredFields()`：获取当前类的所有field（不包括父类）


### 如何使用Field操作成员变量
1. 设置值：`void set(Object obj, Object value) `
2. 获取值：`Object get(Object obj)`
3. 忽略访问权限修饰符的安全检查（暴力反射）：`setAccessible(true)`

## 获取成员方法们
- `Method getMethod(String name, Class<?>... parameterTypes)`：获取某个public的Method（包括父类）
- `Method[] getMethods()`：获取所有public的Method（包括父类）
- `Method getDeclaredMethod(String name, Class<?>... parameterTypes)`：获取当前类的某个Method（不包括父类）
- `Method[] getDeclaredMethods()`：获取当前类的所有Method（不包括父类）

### 如何使用Method执行方法
- `Object invoke(Object obj, Object... args)`：`obj`从中调用底层方法的对象，`args`用于方法调用的参数。
- `String getName()`：获取方法名。


## 获取构造方法们
- `Constructor<T> getConstructor(Class<?>... parameterTypes)`：获取某个public的Constructor
- `Constructor<?>[] getConstructors()`：获取所有public的Constructor
- `Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)`：获取某个Constructor
- `Constructor<?>[] getDeclaredConstructors()`：获取所有Constructor 

### 如何使用Constructor创建对象
- `T newInstance(Object... initargs)`：`initargs`将作为变量传递给构造方法调用的对象数组；基本类型的值被包装在适当类型的包装器对象（如 Float 中的 float）中。

- 如果使用空参数构造方法创建对象，操作可以简化：Class对象的newInstance方法。


## 获取全类名
- `String getName()`

