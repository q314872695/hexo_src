---
title: java线程池的简单使用
tags: java
abbrlink: 3745afed
date: 2020-05-11 21:38:35
---

# 线程池的概念
线程池就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。

由于线程池中有很多操作都是与优化资源相关的，我们在这里就不多赘述。我们通过一张图来了解线程池的工作原理：

![](https://halo-1257208482.image.myqcloud.com/202204051747384.png!webp)

**合理利用线程池能够带来三个好处：**

1. 降低资源消耗。减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
2. 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. 提高线程的可管理性。可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

# 线程池的使用
Java里面线程池的顶级接口是`java.util.concurrent.Executor`，但是严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`java.util.concurrent.ExecutorService`。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，生成一些常用的线程池。官方建议使用Executors工程类来创建线程池对象。

Executors类中有个创建线程池的方法如下：

* `public static ExecutorService newFixedThreadPool(int nThreads)`：返回线程池对象。(创建的是有界线程池,也就是池中的线程个数可以指定最大数量)

获取到了一个线程池ExecutorService 对象，那么怎么使用呢，在这里定义了一个使用线程池对象的方法如下：

- `public Future<?> submit(Runnable task)`:获取线程池中的某一个线程对象，并执行 

> Future接口：用来记录线程任务执行完毕后产生的结果。线程池创建与使用。

**使用线程池中线程对象的步骤：**

1. 创建线程池对象。
2. 创建Runnable接口子类对象。(task)
3. 提交Runnable接口子类对象。(take task)
4. 关闭线程池(一般不做)。

**Runnable实现类代码：**
```java
public class ThreadPoolDemo {
    public static void main(String[] args) {
	// 创建线程池对象
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        executorService.submit(new RunnableImp());
        executorService.submit(new RunnableImp());
        executorService.submit(new RunnableImp());

        executorService.shutdown(); //关闭线程池
        System.out.println("程序结束");

    }
}

class RunnableImp implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName()+"正在执行,"+i);
        }
    }
}

程序结束
pool-1-thread-1正在执行,0
pool-1-thread-1正在执行,1
pool-1-thread-1正在执行,2
pool-1-thread-1正在执行,3
pool-1-thread-1正在执行,4
pool-1-thread-2正在执行,0
pool-1-thread-2正在执行,1
pool-1-thread-2正在执行,2
pool-1-thread-2正在执行,3
pool-1-thread-2正在执行,4
pool-1-thread-3正在执行,0
pool-1-thread-3正在执行,1
pool-1-thread-3正在执行,2
pool-1-thread-3正在执行,3
pool-1-thread-3正在执行,4
```
主线程和线程池中的子线程各执行各的，主线程并不会等待子线程。