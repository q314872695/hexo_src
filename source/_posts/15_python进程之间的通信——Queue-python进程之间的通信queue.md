---
title: python进程之间的通信——Queue
tags: python
abbrlink: 6031ce39
date: 2019-08-22 15:24:00
---



我们知道进程之间的数据是互不影响的，但有时我们需要在进程之间通信，那怎么办呢？

# 认识Queue
可以使用multiprocessing模块的Queue实现多进程之间的数据传递，Queue本身是一个消息列队程序，首先用一个小实例来演示一下Queue的工作原理：
put:
```python
from multiprocessing import Queue
# 创建一个实例，指定最大容量为3，若不指定则无限大(直到内存的尽头)。
q = Queue(3)
q.put("a")
q.put("b")
q.put("c")
# 队列已满，此时程序将被阻塞（停在写入状态），直到从消息列队腾出空间为止。
q.put("d")
```
get:
```python
from multiprocessing import Queue
# 创建一个实例，指定最大容量为3，若不指定则无限大(直到内存的尽头)。
q = Queue(3)
q.put("a")
q.put("b")
q.put("c")
q.get() # 'a'
q.get() # 'b'
q.get() # 'c'
# # 队列为空，此时程序被阻塞，知道队列中再有数据。
q.get()
```
**说明：**
-  **get**(self, block=True, timeout=None) 和 **put**(self, obj, block=True, timeout=None)
1. get和put在默认情况是block(阻塞)为True，timeout(超时时间)=None，只要队列中没有数据或者空队列时一直被阻塞。
2. block设为False(关闭阻塞)，timeout保持默认时，只要队列中没有数据或队满就立即报异常。
get(False)和get_nowait()是等价的，put(要入的的数据,False)和put_nowait(要入队的数据)也是等价的。
3. block=False，timeout=2（timeout超时时间的单位是秒） 表示队满或者队空时，等待2s，如果还是队满或队空，那就报异常。表现形式为：get(False,2)、put(要入队的数据,False,2)

# 使用Queue
我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：
```python
from multiprocessing import Process, Queue
import os
import time
import random


# 写数据进程执行的代码
def write(q):
    for value in ["A", "B", "C"]:
        print("Put %s to queue.." % value)
        q.put(value)
        time.sleep(random.random())


# 读数据进程执行的代码
def read(q):
    while True:
        if not q.empty():
            value = q.get()
            print("Get %s to queue.." % value)
            time.sleep(random.random())
        else:
            break


if __name__ == '__main__':
    # 父进程创建Queue,传给各个子进程
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程
    pw.start()
    # 等待写数据的子进程结束
    pw.join()

    pr.start()
    pr.join()

    print("所有数据都写完并且读完")
```

# 进程池中的Queue
如果要使用Pool创建进程，就需要使用multiprocessing.Manager()中的Queue()，而不是multiprocessing.Queue()，否则会得到一条如下的错误信息：

RuntimeError: Queue objects should only be shared between processes through inheritance.
```python
from multiprocessing import Pool, Manager
import os
import time
import random


# 写数据进程执行的代码
def write(q):
    for value in ["A", "B", "C"]:
        print("Put %s to queue.." % value)
        q.put(value)
        time.sleep(random.random())


# 读数据进程执行的代码
def read(q):
    while True:
        if not q.empty():
            value = q.get()
            print("Get %s to queue.." % value)
            time.sleep(random.random())
        else:
            break


if __name__ == '__main__':
    print("(%s) start" % os.getpid())
    # 父进程创建Queue,传给各个子进程
    q = Manager().Queue()
    po = Pool()
    po.apply(write, (q,))
    po.apply(read, (q,))
    po.close()
    # po.join() # 阻塞式一般不需要

    print("(%s) end" % os.getpid())
```
