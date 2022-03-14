---
title: python中几个常见的魔法方法
date: 2019-08-19 15:24:00
tags: python
---



首先，什么是魔法方法呢？在python中方法名如果是__xxxx__()的，那么就有特殊的功能，因此叫做“魔法”方法。

# __ init__()方法

- 当一个实例被创建的时候调用的初始化方法，在创建对象时默认调用。
- __ init __()方法中默认有一个参数名字为self，如果在创建对象时传递了2个参数，那么__init __()方法除了self作为第一个形参外还需要2个形参，例如__init __(self,x,y)。


**之前我们是这样给对象添加属性的：**
```python
class Student:
	pass
	
stu1 = Student()

stu1.name = "张三"
stu1.age = 18
```
**现在我们利用__init__()方法简化代码**

```python
class Student:
	def __init__(self, name, age):
		self.name = name
		self.age = age
	
stu1 = Student("张三", 18)
```
是不是代码看起来简洁多了呢
# __str __()方法
- 一般用于说明对象，或者自己定义一个想要输出的结果。
- 当调用str()时会调用__str __()，即该对象被强制转换成字符串类型。
- 当使用print()输出该对象时也会调用__str __()方法，只要自己定义了__str __()方法，那么就会打印这个方法中return中的数据。

**没有定义__str __()方法时:**
```python
class Student:
	def __init__(self, name, age):
		self.name = name
		self.age = age
	
stu1 = Student("张三", 18)
print(stu1)

s = str(stu1)
print(s)

"""
输出结果：
<__main__.Student object at 0x03C3BCD0>
<__main__.Student object at 0x03C3BCD0>
"""
```
没有定义__str __()方法时，它默认返回该对象的内存地址。
**定义了__str __()方法是这样的:**
```python
class Student:
	def __init__(self, name, age):
		self.name = name
		self.age = age
	def __str__(self):
		return "姓名：%s\t年龄：%d"%(self.name, self.age)
	
stu1 = Student("张三", 18)
print(stu1)

s = str(stu1)
print(s)

"""
输出结果：
姓名：张三	年龄：18
姓名：张三	年龄：18
"""
```

# __del __()方法
当删除一个对象时，python解释器会默认调用一个方法，这个方法为__del__()方法。
首先应该先了解一个概念，那就是对象的引用个数。我们需要sys模块中的getrefcount()用来测量一个对象的引用个数，**返回值=实际的引用个数+1**。若返回2则说明该对象的实际引用个数为1,此时有1个变量引用了该对象。
```python
import sys
class A:
	pass
a = A()
# 现在只有变量a引用了A类创建的对象
print(sys.getrefcount(a))
"""
输出结果：
2
"""
# 那么现在再创建一个变量b，也引用a所引用的对象，那么它的引用个数就会加1，实际引用个数变成2.
b = a
print(sys.getrefcount(a))
"""
输出结果：
3
"""
```
当python解释器检测到，这个对象的实际引用个数为0时，就会删除这个对象，此时也就会相应的调用__del __()方法。还有一种情况就是该程序已经全部执行完了，那么相应的内存会被释放掉，它也会执行__del __()方法。
**这是程序正常执行完的情况：**
```python
import sys
class A:
	def __del__(self):
		print("该对象被销毁")
a = A()
"""
输出结果：
该对象被销毁
"""
```
**还有一种是手动删除变量引用的情况：**

```python
import sys
class A:
	def __del__(self):
		print("该对象被销毁")
a = A() # 此时实际引用个数为1
b = a # 此时实际引用个数为2

print("删除了变量a") 
del a # 删除变量a，此时实际引用个数为1

print("删除了变量b") 
del b # 删除变量b，此时实际引用个数为0,python解释器就会删除该对象，即调用__del __()方法
print("程序结束")
"""
输出结果：
删除了变量a
删除了变量b
该对象被销毁
程序结束
"""
```
# __new __()方法
- __new __也是类在创建实例时调用的方法，它比__init __调用的时间还早。
- __new __至少要有一个参数cls，代表要实例化的类，此参数在实例化时由python解释器自动提供。
- __new __必须要有返回值，返回实例化出来的实例，这点在自己实现__new__时要特别注意，可以return父类__new__出来的实例，或者直接是object的__new__出来的实例。在Python3中每个类都默认继承的object父类。
- __init__有一个参数self，就是这个__new__返回的实例，__init__在__new__的基础上可以完成一些其它初始化的动作，__init__不需要返回值
```python
class A:
    def __init__(self):
        print("调用了init方法")

    def __new__(cls):
        print("调用了new方法")
        return super().__new__(cls)
a = A()

"""
输出结果：
调用了new方法
调用了init方法
"""
```
## 拓展：可以通过重写__new __方法，实现一个单例模式
代码如下：
```python
class A:
	# 定义一个私有的类属性，用于存储实例化出来的对象
    _isinstance = None
    
    def __new__(cls):
        print("调用了new方法")
        # 判断如果_isinstance为None，则创建一个实例，否则直接返回_isinstance
        if not cls._isinstance:
            cls._isinstance = super().__new__(cls)
        return cls._isinstance


print(id(A))
print(id(A))
print(id(A))
print(id(A))
"""
输出结果：
19888488
19888488
19888488
19888488
"""
```
# __slots __属性
我们都知道python是一门动态语言，可以在程序运行的过程中添加属性。如果我们想要限制实例的属性该怎么办？例如，只允许对Person实例添加name和age属性。
为了达到限制的⽬的，Python允许在定义class的时候，定义⼀个特殊的 __slots__变量，来限制该class实例能添加的属性：
```python
class Person(object):
	__slots__ = ("name", "age") 
P = Person() 
P.name = "⽼王" 
P.age = 20 
P.score = 100 

"""
输出结果：
Traceback (most recent call last):
  File "<input>", line 6, in <module>
AttributeError: 'Person' object has no attribute 'score'
"""
```
注意：使⽤__slots__要注意，__slots__定义的属性仅对当前类实例起作⽤，对 继承的⼦类是不起作⽤的。
