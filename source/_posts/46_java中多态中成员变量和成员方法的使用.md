---
title: java中多态中成员变量和成员方法的使用
tags: java
abbrlink: 8f0eb1a0
date: 2020-05-07 11:03:07
---

# 成员变量的访问规则
- 直接通过对象名称访问成员变量：看等号左边是谁优先用谁，没有则向上找
例如：

```java
class Fu{
	int num = 10;
}
class Zi extends Fu{
	int num = 20;
	int age = 18;
}
public class Main{
	public static void main(String[] args){
		Fu obj = new Zi();
		System.out.println(obj.num); //10
		System.out.println(obj.age); //错误，因为Fu类中没有age变量
	}
}
```

- 间接通过成员方法访问成员变量：看该方法属于谁，优先用谁，没有则向上找

例如：
```java
class Fu{
	int num = 10;
	public void show(){
		System.out.println(num);
	}
}
class Zi extends Fu{
	int num = 20;
	
//	@Override
//	public void show(){
//		System.out.println(num);
//	}
}
public class Main{
	public static void main(String[] args){
		Fu obj = new Zi();
		System.out.println(obj.show()); 
//输出10 子类没有重写，调用父类的show方法，即访问父类的成员变量,当子类重写父类的show()方法时，则访问子类的成员变量，输出20
	}
}
```
# 成员方法的访问规则
- 编译看等号左边，运行看等号右边new的是谁就用谁，没有则向上找
```java
class Fu{
	public void method(){
		System.out.println("父类方法");
	}
	public void methodFu(){
		System.out.println("父类特有方法");
	}
}
class Zi extends Fu{
	@Override
	public void method(){
		System.out.println("子类方法");
	}
	public void methodZi(){
		System.out.println("子类特有方法");
	}
}
public class Main{
	public static void main(String[] args){
		Fu obj = new Zi();
		System.out.println(obj.method()); //子类父类都有优先用子类的
		System.out.println(obj.methodFu()); //子类没有，用父类的
		System.out.println(obj.methodZi()); //Fu类没有methodZi方法编译错误
	}
}
```