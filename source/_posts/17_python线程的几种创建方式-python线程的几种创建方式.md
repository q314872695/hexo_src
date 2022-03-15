---
title: python线程的几种创建方式
tags: python
abbrlink: c61557f1
date: 2019-08-23 15:24:00
---



Python3 线程中常用的两个模块为：

- _thread
- threading(推荐使用)
# 使用Thread类创建
```python
import threading
from time import sleep,ctime

def sing():
    for i in range(3):
        print("正在唱歌...%d"%i)
        sleep(1)

def dance():
    for i in range(3):
        print("正在跳舞...%d"%i)
        sleep(1)

if __name__ == '__main__':
    print('---开始---:%s'%ctime())

    t1 = threading.Thread(target=sing)
    t2 = threading.Thread(target=dance)

    t1.start()
    t2.start()

    #sleep(5) # 屏蔽此行代码，试试看，程序是否会立马结束？
    print('---结束---:%s'%ctime())
"""
输出结果：
---开始---:Sat Aug 24 08:44:21 2019
正在唱歌...0
正在跳舞...0---结束---:Sat Aug 24 08:44:21 2019
正在唱歌...1
正在跳舞...1
正在唱歌...2
正在跳舞...2
"""
```
说明：主线程会等待所有的子线程结束后才结束

# 使用Thread子类创建
为了让每个线程的封装性更完美，所以使用threading模块时，往往会定义一个新的子类class，只要继承threading.Thread就可以了，然后重写run方法。
```python
import threading
import time

class MyThread(threading.Thread):
    def run(self):
        for i in range(3):
            time.sleep(1)
            msg = "I'm "+self.name+' @ '+str(i) #name属性中保存的是当前线程的名字
            print(msg)


if __name__ == '__main__':
    t = MyThread()
    t.start()
"""
输出结果：
I'm Thread-5 @ 0
I'm Thread-5 @ 1
I'm Thread-5 @ 2
"""
```
# 使用线程池ThreadPoolExecutor创建
```python
from concurrent.futures import ThreadPoolExecutor
import time
import os


def sayhello(a):
    for i in range(10):
        time.sleep(1)
        print("hello: " + a)


def main():
    seed = ["a", "b", "c"]
    # 最大线程数为3，使用with可以自动关闭线程池，简化操作
    with ThreadPoolExecutor(3) as executor:
        for each in seed: 
        	# map可以保证输出的顺序, submit输出的顺序是乱的
            executor.submit(sayhello, each)

    print("主线程结束")


if __name__ == '__main__':
    main()
```
