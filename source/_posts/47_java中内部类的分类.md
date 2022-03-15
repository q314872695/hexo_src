---
title: java中内部类的分类
tags: java
abbrlink: 6d1855f0
date: 2020-05-07 20:02:56
---

权限修饰符：
public > protected > (default) > private

**定义一个类的时候，权限修饰符规则：**
1. 外部类：public / (default)
2. 成员内部类：public / protected / (default) / private
3. 局部内部类：什么都不能写
# 成员内部类
定义格式：
```java
修饰符 class 外部类名称{

	修饰符 class 内部类名称{
		...
	}
	...
}
```
注意：内用外，随意访问；外用内，需要内部类对象

如何使用成员内部类？
1. 间接方式：在外部类的方法中使用内部类，然后main()只调用外部类的方法。
```java
public class Body { // 外部类

    public class Heart { // 成员内部类

        // 内部类的方法
        public void beat() {
            System.out.println("心脏跳动：蹦蹦蹦！");
            System.out.println("我叫：" + name); // 正确写法！
        }

    }

    // 外部类的成员变量
    private String name="张三";

    // 外部类的方法
    public void methodBody() {
        System.out.println("外部类的方法");
        new Heart().beat(); //外部类方法使用内部类
    }
}
public class Demo01InnerClass {
    public static void main(String[] args) {
        Body body = new Body(); // 外部类的对象
        body.methodBody();
    }
}
```

2. 直接方式：`外部类名称.内部类名称  对象名 = new  外部类名称().new  内部类名称()`
```java
public class Demo01InnerClass {
    public static void main(String[] args) {
        Body.Heart heart = new Body().new Heart();
        heart.beat();
    }
}
```

**内部类和外部类成员变量重名的问题**

当成员内部类拥有和外部类同名的成员变量或者方法时，会发生隐藏现象，即默认情况下访问的是成员内部类的成员。如果要访问外部类的同名成员，需要以下面的形式进行访问：

`外部类.this.成员变量`
`外部类.this.成员方法`

例如：
```java
public class Outer {
    int num = 10; // 外部类的成员变量

    public class Inner{

        int num = 20; // 内部类的成员变量

        public void methodInner() {
            int num = 30; // 内部类方法的局部变量
            System.out.println(num); // 局部变量，就近原则
            System.out.println(this.num); // 内部类的成员变量
            System.out.println(Outer.this.num); // 外部类的成员变量
        }

    }

}
```
# 局部内部类
如果一个类定义在一个方法的内部，那么这就是一个局部内部类。
只有当前所属的方法才能访问它，出了这个方法就不能用了。

定义格式：
```java
修饰符 class 外部类名称 {
    修饰符 返回值类型 外部类方法名称(参数列表) {
        class 局部内部类名称 {
            // ...
        }
    }
}
```
如果希望访问所在方法的局部变量，那么这个局部变量必须是**有效final的**
从Java 8+开始，只要局部变量事实不变，那么final关键字可以省略。

例如：
```java
public class MyOuter {

    public void methodOuter() {
        int num = 10; // 所在方法省略final的局部变量

        class MyInner {
            public void methodInner() {
                System.out.println(num);
            }
        }
    }

}

```
原因：
1. new出来的对象在堆内存当中。
2. 局部变量是跟着方法走的，在栈内存当中。
3. 方法运行结束之后，立刻出栈，局部变量就会立刻消失。
4. 但是new出来的对象会在堆当中持续存在，直到垃圾回收消失。 


# 匿名内部类
如果接口的实现类（或者是父类的子类）只需要使用唯一的一次，
那么这种情况下就可以省略掉该类的定义，而改为使用**匿名内部类**。

定义格式：
```java
接口名称 对象名 = new 接口名称() {
    // 覆盖重写所有抽象方法
};
```
对格式`new 接口名称() {...}`进行解析：
1. new代表创建对象的动作
2. 接口名称就是匿名内部类需要实现哪个接口
3. {...}这才是匿名内部类的内容

例如：
```java
public interface MyInterface {

    void method1(); // 抽象方法
}
public class DemoMain {

    public static void main(String[] args) {

        // 使用匿名内部类，但不是匿名对象，对象名称就叫objA
        MyInterface objA = new MyInterface() {
            @Override
            public void method1() {
                System.out.println("匿名内部类实现了方法！");
            }
        };
        objA.method1();

        // 使用了匿名内部类，而且省略了对象名称，也是匿名对象
        new MyInterface() {
            @Override
            public void method1() {
                System.out.println("匿名内部类实现了方法！");
            }
        }.method1();
}
```
注意事项：
1. 匿名内部类，在【创建对象】的时候，只能使用唯一一次。
如果希望多次创建对象，而且类的内容一样的话，那么就需要使用单独定义的实现类了。
2. 匿名对象，在【调用方法】的时候，只能调用唯一一次。
如果希望同一个对象，调用多次方法，那么必须给对象起个名字。
3. 匿名内部类是省略了【实现类/子类名称】，但是匿名对象是省略了【对象名称】


强调：匿名内部类和匿名对象不是一回事！！！

