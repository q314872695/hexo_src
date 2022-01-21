---
title: java中String和StringBuffer的区别
date: 2020-03-31 14:21:32
tags: java
---


String和StringBuffer的区别

<!--more-->

### String

- **String类对象一旦创建就不可更改**。
- String对象表示不可修改的Unicode编码字符串。
- Java中双引号括起来的字符串也被当做String对象。

例如：
```java
System.out.println("abc".length()); // 输出3
```
### StringBuffer
- **StringBuffer对象的内容是可以改变的**。
- 如果经常对字符串内容进行修改，则使用StringBuffer。
- 如果经常对String对象字符串内容进行修改的话，就会导致即耗时间又耗空间。
- StringBuffer类中有大量修改字符串的方法。

创建一个StringBuffer字符串对象：
```java
StringBuffer stringBuffer = new StringBuffer("abc");
stringBuffer.append("d"); // 字符串末尾附加d
System.out.println(stringBuffer); // 输出：abcd
```
### StringBuilder
- 和`StringBuffer`是类似的，只是它的线程不安全的，但速度比`StringBuffer`更快