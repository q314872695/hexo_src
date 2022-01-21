---
title: java中properties属性集介绍
date: 2020-05-13 21:13:03
tags: java
---

# 概述

`java.util.Properties ` 继承于` Hashtable` ，来表示一个持久的属性集。它使用键值结构存储数据，每个键及其对应值都是一个字符串。该类也被许多Java类使用，比如获取系统属性时，`System.getProperties` 方法就是返回一个`Properties`对象。

# Properties类

## 构造方法

- `public Properties()` :创建一个空的属性列表。

## 基本的存储方法

- `public Object setProperty(String key, String value)` ： 保存一对属性。  
- `public String getProperty(String key) ` ：使用此属性列表中指定的键搜索属性值。
- `public Set<String> stringPropertyNames() ` ：所有键的名称的集合。
- `public void load(Reader reader)`： 从字符输入流中读取键值对。
- `public void store(Writer writer, String comments)`：将此 Properties 表中的属性列表（键和元素对）写入字符输出流中。`comments`为属性列表的描述，一般为空字符串即可。
## 实例
**把Properties输出到指定文件中**
```java
public class PropertiesDemo {
    public static void main(String[] args) throws IOException {
        Properties properties = new Properties();
        properties.setProperty("name", "张三");
        properties.setProperty("age", "18");
        properties.store(new FileWriter(".\\newFile.txt"), "");
    }
}
newFile.txt内容
#
#Wed May 13 21:12:54 CST 2020
age=18
name=张三
```
**从指定文件中读取属性**
```java
public class PropertiesDemo {
    public static void main(String[] args) throws IOException {
        Properties properties = new Properties();
        properties.load(new FileReader(".\\newFile.txt"));
        Set<String> strings = properties.stringPropertyNames();
        for (String key : strings) {
            System.out.println(key+"="+properties.getProperty(key));
        }
    }
}
控制台内容：
age=18
name=张三
```