---
title: python多任务——协程的使用
date: 2019-08-26 15:24:00
tags: python
---



# 使用yield完成多任务

```python
import time

def test1():
    while True:
        print("--1--")
        time.sleep(0.5)
        yield None

def test2():
    while True:
        print("--2--")
        time.sleep(0.5)
        yield None

if __name__ == "__main__":
        t1 = test1()
        t2 = test2()
        while True:
                next(t1)
                next(t2)
```
# 使用greenlet完成多任务
如果没有安装，则**pip install greenlet**
```python
from greenlet import greenlet
import time


def test1():
    while True:
        print("---A---")
        gr2.switch()
        time.sleep(0.5)


def test2():
    while True:
        print("---b---")
        gr1.switch()
        time.sleep(0.5)


gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch()
```
# 使用gevent完成多任务
首先使用**pip install gevent**进行安装
gevent是对greenlet的再次封装，使用起来更加简便，当有耗时操作时会自动切换到其他协程。gevent封装了常用的耗时操作，如thread、socket、time、multiprocessing等模块。
```python
import gevent
import time
from gevent import monkey
# 打补丁，不需要改原来的耗时操作，否则是要使用gevent提供的模块
monkey.patch_all()
def f(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        # 使用gevent提供的耗时模块
        # gevent.sleep(0.5)
        time.sleep(0.5)


g1 = gevent.spawn(f, 5)
g2 = gevent.spawn(f, 5)
g3 = gevent.spawn(f, 5)
# g1.join()
# g2.join()
# g3.join()
# 一次性添加全部的任务
gevent.joinall([g1,g2,g3])

```
