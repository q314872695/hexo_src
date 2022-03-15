---
title: java案例：计算一个人已经出生多少天
tags: java
abbrlink: 6818d085
date: 2020-05-08 16:19:11
---

# 需要的知识点
## Date类
**常用构造函数**
- `public Date()`：使用当前日期和时间来初始化对象。
- `public Date(long date)`：接收一个参数，该参数是从1970年1月1日起的毫秒数


**常用方法**
- `public long getTime()` 把日期对象转换成对应的时间毫秒值。例如获取当前时间的毫秒值可以使用`new Date().getTime()`,或者也可以使用`System.currentTimeMillis()`.
## DateFormat类
`java.text.DateFormat` 是日期/时间格式化子类的抽象类，我们通过这个类可以帮我们完成日期和文本之间的转换,也就是可以在Date对象与String对象之间进行来回转换。

**构造方法**

由于DateFormat为抽象类，不能直接使用，所以需要常用的子类`java.text.SimpleDateFormat`。这个类需要一个模式（格式）来指定格式化或解析的标准。构造方法为：
- `public SimpleDateFormat(String pattern)`：用给定的模式和默认语言环境的日期格式符号构造`SimpleDateFormat`。

常用的格式规则：

|标识字母（区分大小写）|含义|
|-------|-------|
|y|年|
|M|月|
|d|日|
|H|时|
|m|分|
|s|秒|

**常用方法**
- `public String format(Date date)`：格式化，按照指定的格式，从Date对象转换为String对象。
- `public Date parse(String source)`：解析，按照指定的格式，从String对象转换为Date对象。


# 分析
1. 使用`Scanner`类中的方法`next()`获取键盘输入的出生日期
2. 使用`SimpleDateFormat`类中的方法`parse()`方法把字符串的出生日期解析为Date格式
3. 把Date格式的出生日期转换成毫秒值
4. 获取当前时间转换成毫秒值
5. 使用当前日期的毫秒值-出生日期的毫秒值
6. 把毫秒差值转换为天(s/1000/60/60/24)

# 实现
```java
public class DemoDate {
    public static void main(String[] args) throws ParseException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入您的出生日期（格式为：yyyy-MM-dd）：");
        String birthdayDate=sc.next();
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        Date date = simpleDateFormat.parse(birthdayDate);
        long birthdayTime = date.getTime();
        long todayTime=System.currentTimeMillis();
        long time = todayTime - birthdayTime;
        System.out.println("您已经出生"+time/1000/3600/24+"天");
    }
}
```
