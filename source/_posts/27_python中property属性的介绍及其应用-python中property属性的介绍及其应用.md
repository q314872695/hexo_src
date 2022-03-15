---
title: python中property属性的介绍及其应用
tags: python
abbrlink: 5366f359
date: 2019-08-28 15:24:00
---



Python的property属性的功能是：property属性内部进行一系列的逻辑计算，最终将计算结果返回。
使用property修饰的实例方法被调用时，可以把它当做实例属性一样

# property的用法1——装饰器方式
在类的实例方法上应用@property装饰器
```python
class Test:
    def __init__(self):
        self.__num = 100

    @property
    def num(self):
        print("--get--")
        return self.__num

    @num.setter
    def num(self, num):
        print("--set--")
        self.__num = num

t = Test()
print(t.num)
t.num = 1
"""
--get--
100
--set--
"""
```
**property属性的定义和调用要注意一下几点：**
- 定义时，在实例方法的基础上添加 @property 装饰器；并且仅有一个self参数。
- 调用时，无需括号。
- 经典类中的属性只有一种访问方式，其对应被 @property 修饰的方法。
- 新式类中的属性有三种访问方式，并分别对应了三个被@property、@方法名.setter、@方法名.deleter修饰的方法。
- Python中的类有经典类和新式类，新式类的属性比经典类的属性丰富。（ 如果类继object，那么该类是新式类 ），python3中的类都是新式类。
# property的用法2——类属性方式
当使用类属性的方式创建property属性时，经典类和新式类无区别
```python
class Test:
    def __init__(self):
        self.__num = 100

    def setNum(self, num):
        print("--set--")
        self.__num = num

    def getNum(self):
        print("--get--")
        return self.__num

    # 注意：要先写get方法，再写set方法
    aa = property(getNum, setNum)


t = Test()
print(t.aa)
t.aa = 1
```
