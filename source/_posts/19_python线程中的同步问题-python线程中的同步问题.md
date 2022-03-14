---
title: python线程中的同步问题
date: 2019-08-24 15:24:00
tags: python
---



# 多线程开发可能遇到的问题

假设两个线程t1和t2都要对num=0进行增1运算，t1和t2都各对num修改1000000次，num的最终的结果应该为2000000。但是由于是多线程访问，有可能出现下面情况：
```python
from threading import Thread
import time

num = 0

def test1():
    global num
    for i in range(1000000):
        num += 1

    print("--test1--num=%d" % num)


def test2():
    global num
    for i in range(1000000):
        num += 1

    print("--test2--num=%d" % num)


if __name__ == '__main__':
    Thread(target=test1).start()
    Thread(target=test2).start()
    print("num = %d" % num)
"""
num = 134116
--test1--num=1032814
--test2--num=1166243
"""
```
运行结果可能不一样，但是结果往往不是2000000。问题产生的原因就是没有控制多个线程对同一资源的访问，对数据造成破坏，使得线程运行的结果不可预期。这种现象称为“线程不安全”。

# 线程同步——使用互斥锁
如果多个线程共同对某个数据修改，则可能出现不可预料的结果，为了保证数据的正确性，需要对多个线程进行同步。
使用 Thread 对象的 Lock 和 Rlock 可以实现简单的线程同步，这两个对象都有 acquire 方法和 release 方法，对于那些需要每次只允许一个线程操作的数据，可以将其操作放到 acquire 和 release 方法之间。

**使用互斥锁实现上面的例子：**
```python
from threading import Thread, Lock
import time

num = 0


def test1():
    global num
    # 上锁
    mutex.acquire()
    for i in range(1000000):
        num += 1
    # 解锁
    mutex.release()
    print("--test1--num=%d" % num)


def test2():
    global num
    mutex.acquire()
    for i in range(1000000):
        num += 1
    mutex.release()
    print("--test2--num=%d" % num)


start_time = time.time()  # 开始时间
# 创建一把互斥锁，默认没有上锁
mutex = Lock()
p1 = Thread(target=test1)
p1.start()

# time.sleep(3)   # 取消屏蔽之后 再次运行程序，结果会不一样，，，为啥呢？

p2 = Thread(target=test2)
p2.start()
p1.join()
p2.join()
end_time = time.time()  # 结束时间
print("num = %d" % num)

print("运行时间:%fs" % (end_time - start_time))  # 结束时间-开始时间

"""
输出结果：
--test1--num=1000000
--test2--num=2000000
num = 2000000
运行时间:0.287206s
"""
```


# 同步的应用——多个线程有序执行
```python
from threading import Lock, Thread
from time import sleep


class Task1(Thread):
    def run(self):
        while True:
        	# 判断是否上锁成功，返回值为bool类型
            if lock1.acquire():
                print("--task1--")
                sleep(0.5)
                lock2.release()


class Task2(Thread):
    def run(self):
        while True:
            if lock2.acquire():
                print("--task2--")
                sleep(0.5)
                lock3.release()


class Task3(Thread):
    def run(self):
        while True:
            if lock3.acquire():
                print("--task3--")
                sleep(0.5)
                lock1.release()

if __name__ == '__main__':    
    # 创建一把锁
    lock1 = Lock()
    
    # 创建一把锁，并且锁上
    lock2 = Lock()
    lock2.acquire()
    
    # 创建一把锁，并且锁上
    lock3 = Lock()
    lock3.acquire()
    
    t1 = Task1()
    t2 = Task2()
    t3 = Task3()
    
    t1.start()
    t2.start()
    t3.start()
"""
--task1--
--task2--
--task3--
--task1--
--task2--
--task3--
--task1--
--task2--
...
"""
```

# 生产者与消费者模式
## 为什么要使用生产者和消费者模式
在线程世界里，生产者就是生产数据的线程，消费者就是消费数据的线程。在多线程开发当中，如果生产者处理速度很快，而消费者处理速度很慢，那么生产者就必须等待消费者处理完，才能继续生产数据。同样的道理，如果消费者的处理能力大于生产者，那么消费者就必须等待生产者。为了解决这个问题于是引入了生产者和消费者模式。
## 什么是生产者消费者模式
生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。

Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语（可以理解为原子操作，即要么不做，要么就做完），能够在多线程中直接使用。可以使用队列来实现线程间的同步。

用FIFO队列实现上述生产者与消费者问题的代码如下：
```python
import threading
import time
from queue import Queue


class Producer(threading.Thread):
    def run(self):
        global queue
        count = 0
        while True:
            if queue.qsize() < 1000:
                for i in range(100):
                    count += 1
                    msg = "生成产品" + str(count)
                    queue.put(msg)
                    print(msg)
            time.sleep(0.5)


class Consumer(threading.Thread):
    def run(self):
        global queue
        while True:
            if queue.qsize() > 100:
                for i in range(3):
                    msg = self.name + "消费了" + queue.get()
                    print(msg)
            time.sleep(0.5)


if __name__ == '__main__':
    queue = Queue()

    for i in range(500):
        queue.put("初始产品" + str(i))
    # 创建2个生产线程
    for i in range(2):
        p = Producer()
        p.start()
    # 创建5个消费线程
    for i in range(5):
        c = Consumer()
        c.start()
```

# ThreadLocal
在多线程环境下，每个线程都有自己的数据。一个线程使用自己的局部变量比使用全局变量好，因为局部变量只有线程自己能看见，不会影响其他线程，而全局变量的修改必须加锁。
**ThreadLocal解决了参数在一个线程中各个函数之间互相传递的问题**
```python
import threading
"""
⼀个ThreadLocal变量虽然是全局变量，但每个线程都只能读写⾃⼰线程的独
⽴副本，互不⼲扰。
"""
# 创建全局ThreadLocal对象:
local_school = threading.local()


def process_student():
    # 获取当前线程关联的student:
    std = local_school.student
    print('Hello, %s (in %s)' % (std, threading.current_thread().name))


def process_thread(name):
    # 绑定ThreadLocal的student:
    local_school.student = name
    process_student()


t1 = threading.Thread(target=process_thread, args=('dongGe',), name="Thread-A")
t2 = threading.Thread(target=process_thread, args=('⽼王',), name="Thread-B")
t1.start()
t2.start()
```


