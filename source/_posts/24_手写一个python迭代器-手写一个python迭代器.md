---
title: 手写一个python迭代器
date: 2019-08-26 15:24:00
tags: python
---



# 分析

- 我们都知道一个可迭代对象可以通过iter()可以返回一个迭代器。
- 如果想要一个对象称为可迭代对象，即可以使用for，那么必须实现__iter __()方法。
- 在一个类的实例对象想要变成迭代器，就必须实现__iter__()和__next__()方法。
- 调用iter()时，在对象内部默认调用__iter__()，即__iter__()的返回值应该是一个迭代器。
- for的每次循环中或者next()时，都是自动调用迭代器的__next__()方法，并有一个返回值。
# 实现
```python
class Classmate:
    def __init__(self):
        self.names = []
        self.num = 0

    def add(self, name):
        self.names.append(name)

    def __iter__(self):
        return self

    def __next__(self):
        if self.num < len(self.names):
            ret = self.names[self.num]
            self.num += 1
            return ret
        else:
            raise StopIteration

c = Classmate()
c.add("张三")
c.add("李四")
c.add("王五")
for i in c:
    print(i)
"""
张三
李四
王五
"""
```
