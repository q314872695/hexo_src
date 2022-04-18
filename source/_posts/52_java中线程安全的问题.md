---
title: java中线程安全的问题
tags: java
abbrlink: 8c71e0cd
date: 2020-05-11 21:19:12
---

# 线程安全
如果有多个线程在同时运行，而这些线程可能会同时运行这段代码。程序每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。 

我们通过一个案例，演示线程的安全问题： 

电影院要卖票，我们模拟电影院的卖票过程。假设要播放的电影是 “葫芦娃大战奥特曼”，本次电影的座位共100个 (本场电影只能卖100张票)。 

我们来模拟电影院的售票窗口，实现多个窗口同时卖 “葫芦娃大战奥特曼”这场电影票(多个窗口一起卖这100张票) 

需要窗口，采用线程对象来模拟；需要票，Runnable接口子类来模拟 

模拟票： 
```java
public class Ticket implements Runnable {
    private int ticket = 100;

    @Override
    public void run() {
        while (true) {
            if (ticket > 0) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "正在卖：" + ticket--);
            }
        }
    }
}
```
测试类：
```java
public class TicketDemo {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        new Thread(ticket, "窗口1").start();
        new Thread(ticket, "窗口2").start();
        new Thread(ticket, "窗口3").start();
    }
}
```
结果中有一部分这样现象：

![](https://halo-1257208482.image.myqcloud.com/202204051747005.png!webp)

1. 相同的票数,比如5这张票被卖了两回。 
2. 不存在的票，比如0票与-1票，是不存在的。


这种问题，几个窗口(线程)票数不同步了，这种问题称为线程不安全。

> 线程安全问题都是由全局变量及静态变量引起的。若每个线程中对全局变量、静态变量只有读操作，而无写 操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步， 否则的话就可能影响线程安全。

# 线程同步
为了保证每个线程都能正常执行原子操作,Java引入了线程同步机制。 那么怎么去使用呢？有三种方式完成同步操作：
1. 同步代码块
2. 同步方法
3. 锁机制
## 同步代码块
`synchronized`关键字可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。 

**格式**:
```java
synchronized(同步锁){
	需要同步操作的代码
}
```
**同步锁**：对象的同步锁只是一个概念,可以想象为在对象上标记了一个锁。
1. 锁对象可以使任意类型
2. 多个线程对象要使用同一把锁

> 注意:在任何时候,最多允许一个线程拥有同步锁,谁拿到锁就进入代码块,其他的线程被阻塞。

**使用同步代码块解决代码**：
```java
public class Ticket implements Runnable {
    private int ticket = 100;

    @Override
    public void run() {
        while (true) {
            synchronized (this){ //把当前对象当做一把同步锁
                if (ticket > 0) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "正在卖：" + ticket--);
                }
            }
        }
    }
}
```
## 同步方法
使用`synchronized`修饰的方法,就叫做同步方法,保证A线程执行该方法的时候,其他线程只能在方法外等着。

**格式：**
```java
public synchronized void method(){
	可能会产生线程安全问题的代码
}
```
同步锁是谁：
- 对于非静态方法，同步锁就是this
- 对于静态方法，同步锁就是我们使用当前方法所在类的字节码对象(类名.class)

**使用同步方法代码如下：**
```java
public class Ticket implements Runnable {
    private int ticket = 100;

    @Override
    public void run() {
        while (true) {
            sellTicket();
        }
    }
    public synchronized void sellTicket(){
        if (ticket > 0) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "正在卖：" + ticket--);
        }
    }
}
```
## Lock锁
`java.util.concurrent.locks.Lock`机制提供了比`synchronized`代码块和`synchronized`方法更广泛的锁定操作, 同步代码块/同步方法具有的功能`Lock`都有,除此之外更强大,更体现面向对象。

Lock锁也称同步锁，加锁与释放锁方法化了，如下：
- `public void lock()` :加同步锁。
- `public void unlock()` :释放同步锁。


**使用同步锁的代码如下：**
`Lock`是一个接口，需要借助它的实现类`java.util.concurrent.locks.ReentrantLock`
```java
public class Ticket implements Runnable {
    private int ticket = 100;
    private Lock lock = new ReentrantLock();
    @Override
    public void run() {
        while (true) {
            lock.lock();
            if (ticket > 0) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "正在卖：" + ticket--);
            }
            lock.unlock();
        }
    }
}
```



