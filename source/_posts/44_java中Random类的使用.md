---
title: java中Random类的使用
tags: java
abbrlink: 53cd40e1
date: 2020-05-03 21:54:19
---

>Random类用来生成随机数字
# Random类的使用步骤
1. 导包
```java
import java.util.Random;
```

2. 创建对象
```java
Random r = new Random();
```

3. 使用
```java
int num=r.nextInt();  //随机生成一个整数，这个整数的范围就是int类型的范围-2^31~2^31-1
```
同理，`nextBoolean()`返回布尔类型的随机数，`nextDouble()`返回double类型的随机数...
其中，nextInt(int n)返回指定范围的int类型的随机数，这个范围[0,n)左闭右开，包括0而不包括n的随机数，例如n=3时，它返回的随机数的可能取值的0，1，2。
# 练习

根据int变量n的值，来获取随机数字，范围[1,n]，可以取到1也可以取到n。
```java
int n=5;
Random r=new Random();
int result=r.nextInt(n)+1;
System.out.println(result);
```

# 进阶使用
- Random类中实现的随机算法是伪随机，也就是有规则的随机。在进行随机时，随机算法的起源数字称为种子数( seed)，在种子数的基础上进行一定的变换，从而产生需要的随机数字。
- 相同种子数的 Random对象，相同次数生成的随机数字是完全相同的。也就是说，两个种子数相同的 Random对象，第一次生成的随机数字完全相同，第二次生成的随机数字也完全相同。这点在生成多个随机数字时需要特别注意。

例如：
```java
import java.util.Random;

public class RandomDemo1 {
    public static void main(String[] args) {
        int i = 0;
        int j = 0;
        Random random = new Random(1);
        Random random1 = new Random(1);
        i = random.nextInt();
        j = random1.nextInt();
        System.out.println("i:" + i + "\nj:" + j);
    }
}
/*运行结果：
i:-1155869325
j:-1155869325
* */
```
**再次强调：种子数只是随机算法的起源数字，和生成的随机数字的区间无关。**
## Random类的构造方法
- `public Random()`该构造方法使用一个和当前系统时间对应的相对时间有关的数字作为种子数，然后使用这个种子数构造 Random对象。
- `public Random(long seed)`该构造方法可以通过指定一个种子数进行创建。
## Random的常用示例
1. 想生成范围在[0,n]的整数
```java
random.nextInt(n+1);
```
2. 想生成范围在[m,n]的整数
```java
random.nextInt(n-m+1) + m;
```
3. 生成[0,5.0)区间的小数
```java
random.nextDouble() * 5;
```
4. 生成[1,2.5)区间的小数
```java
random.nextDouble() * 1.5 + 1;//先算出[0,1.5)的随机数，然后整体加1
```




