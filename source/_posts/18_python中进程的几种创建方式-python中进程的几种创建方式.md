---
title: python中进程的几种创建方式
date: 2019-08-23 15:24:00
tags: python
---



**在新创建的子进程中，会把父进程的所有信息复制一份，它们之间的数据互不影响。**

# 使用os.fork()创建
该方式只能用于Unix/Linux操作系统中，在windows不能用。
```python
import os

# 注意，fork函数，只在Unix/Linux/Mac上运行，windows不可以
pid = os.fork()
# 子进程永远返回0，而父进程返回子进程的ID。
if pid == 0:
	print('子进程')
else:
	print('父进程')
```
# 使用Process类类创建
multiprocessing模块提供了一个Process类来代表一个进程对象，下面的例子演示了启动一个子进程并等待其结束：
```python
from multiprocessing import Process
import time

def test(name, age):
    for i in range(5):
        print("--test--%s\t%d" % (name, age))
        time.sleep(1)
    print("子进程结束")


if __name__ == '__main__':
    p = Process(target=test, args=("aaa", 18))
    p.start()
    # 等待进程实例执⾏结束，或等待多少秒；
    p.join() # 等待的最长时间
    print("主进程结束")
"""
输出结果：
--test--aaa	18
--test--aaa	18
--test--aaa	18
--test--aaa	18
--test--aaa	18
子进程结束
主进程结束
"""
```
join()方法表示主进程等待子进程执行完成后继续往下执行，如果把join()注释掉，则主进程开启子进程后不停顿继续往下执行，然后等待子进程完成程序结束。
把join()方法注释掉的结果：
```python
"""
输出结果：
主进程结束
--test--aaa	18
--test--aaa	18
--test--aaa	18
--test--aaa	18
--test--aaa	18
子进程结束
"""
```
# 使用Process子类创建
创建新的进程还能够使用类的方式，可以自定义一个类，继承Process类，每次实例化这个类的时候，就等同于实例化一个进程对象，请看下面的实例：
```python
from multiprocessing import Process
import time
import os


class MyProcess(Process):

    def __init__(self):
        # 如果子类要重写__init__是必须要先调用父类的__init__否则会报错
        # Process.__init__(self)   
        super(MyProcess,self).__init__()

    # 重写Porcess的run()方法
    def run(self):
        print("子进程(%s)开始执行，父进程(%s)" % (os.getpid(), os.getppid()))
        for i in range(5):
            print("--1--")
            time.sleep(1)


if __name__ == '__main__':
    t_start = time.time()
    p = MyProcess()
    p.start()
    # p.join()
    print("main")
    for i in range(5):
        print("--main--")
        time.sleep(1)
```
# 使用进程池Pool创建
当需要创建的子进程数量不多时，可以直接利用multiprocessing中的Process动态成生多个进程，但如果是上百甚至上千个目标，手动的去创建进程的工作量巨大，此时就可以用到multiprocessing模块提供的Pool方法。
初始化Pool时，可以指定一个最大进程数，当有新的请求提交到Pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到指定的最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来执行，请看下面的实例：
```python
from multiprocessing import Pool
import os
import time


def worker(num):
    # for i in range(3):
    print("----pid=%d  num=%d---" % (os.getpid(), num))
    time.sleep(1)

if __name__ == '__main__':
	# 定义一个进程池，最大进程数3
    pool = Pool(3)
    for i in range(10):
        print("---%d--" % i)
        # 使用非阻塞方式调用func（并行执行)，一般用这个。
        # apply堵塞方式必须等待上一个进程退出才能执行下一个进程,用的不多。
        pool.apply_async(worker, (i,))
    # 关闭进程池
    pool.close()
    # 等待所有子进程结束，主进程一般用来等待
    pool.join()  # 进程池后面无操作时必须有这句
```
