---
title: with与上下文管理器
tags: python
abbrlink: 41da7731
date: 2019-08-30 15:24:00
---



# with关键字

在程序中，有很多连接需要关闭和打开，如文件的读写、数据库连接、socket，频繁的手动关闭比较麻烦，就出现的with。
例如对文件的操作正常情况下我们是这样的：
```python
f = open("a.txt", "w")
f.write("python")
f.close()
```
使用with关键字以后：
```python
with open("a.txt", "r") as f:
        f.write("Python")
```
这是一种更加简洁的语法。

# 上下文管理器
任何实现了 __enter __() 和 __exit __() 方法的对象都可称之为上下文管理器，上下文管理器对象可以使用 with 关键字。显然，文件（file）对象也实现了上下文管理器。
__enter __() 方法返回资源对象，这里就是你将要打开的那个文件对象，__exit __() 方法处理一些清除工作。
## 自己实现一个上下文管理器
```python
class File:
    def __init__(self, file_name, mode):
        self.file_name = file_name
        self.mode = mode

    def __enter__(self):
        self.f = open(self.file_name, self.mode, encoding="utf-8")
        print("enter")
        return self.f

    def __exit__(self, *args):
        self.f.close()
        print("exit")


with File("a.txt", "w") as f:
    f.write("你好")
```

## 使用contextmanager 装饰器实现上下文管理器
Python 还提供了一个 contextmanager 的装饰器，更进一步简化了上下文管理器的实现方式。通过 yield 将函数分割成两部分，yield 之前的语句在 __enter__ 方法中执行，yield 之后的语句在 __exit__ 方法中执行。紧跟在 yield 后面的值是函数的返回值。
```python
from contextlib import contextmanager

@contextmanager
def file(file_name, mode):
    f = open(file_name,mode,encoding="utf-8")
    yield f
    f.close()

with file("a.txt", "w") as f:
    f.write("你fdf好")
```
