---
title: python的类属性、实例属性、类方法、静态方法
date: 2019-08-20 15:24:00
tags: python
---



# 类属性

就像如下代码：
```python
class Person:
	name = "张三" # 共有类属性
	__age = 18 # 私有类属性
```
在类中直接定义的属性就是类属性，**它被所有的实例对象所共有**。
**对于共有类属性，在类外可通过类对象和实例对象访问**。
例如：
```python
class Person:
	name = "张三" # 共有类属性
	__age = 18 # 私有类属性

p = Person()
print(p.name) # 通过实例对象访问共有属性
print(Person.name) # 通过类对象访问共有属性
"""
输出结果：
张三
张三
"""
```
**私有的类属性在类外不能被访问，否则会报异常。**

# 实例属性
- 在类中通过self.xxx或者实例对象.xxx定义的属性就是实例属性。
- 每个实例属性仅在当前实例对象中有，若想要所有的实例对象有该实例属性，则需要在__init __初始化方法中定义该实例属性。

如下代码所示定义实例属性：
```python
class Person:
	def __init__(self):
		self.name = "张三" # 定义实例属性
	

p = Person()
print(p.name)
```
或者
```python
class Person:
	pass

p = Person()
p.name = "张三" # 定义实例属性
print(p.name)
```

## 通过类对象修改类属性的值
也许我们会想到通过实例对象去修改类属性的值，那么它是这样的：
```python
class Person:
	sex = "男"
	def __init__(self):
		self.sex = "女"

p = Person()
print(p.sex)
print(Person.sex)
"""
输出结果：
女
男
"""
```
也可能是这样想的：
```python
class Person:
	sex = "男"

p = Person()
p.sex = "女"
print(p.sex)
print(Person.sex)
"""
输出结果：
女
男
"""
```
本质上它们是一样的，并没有修改类属性的值，只是在当前的实例对象中添加了一个和类属性同名实例属性，并且之后如果通过实例对象去引用该名称的属性，实例属性会强制屏蔽掉类属性，即引用的是实例属性，除非删除了该实例属性。
**只有通过类对象才能修改类属性的值**
```python
class Person:
	sex = "男"

Person.sex = "女"
print(Person.sex)
"""
输出结果：
女
"""
```

# 类方法
类对象拥有的方法就是类方法，需要用修饰器@classmethod来标识其为类方法，对于类方法，第一个参数必须是类对象，一般以cls作为第一个参数（当然可以用其他名称的变量作为其第一个参数，但是大部分人都习惯以'cls'作为第一个参数的名字，就最好用'cls'了），**能够通过实例对象和类对象去访问。**
```python
class People(object):
    country = 'china'

    #类方法，用classmethod来进行修饰
    @classmethod
    def getCountry(cls):
        return cls.country

p = People()
print(p.getCountry())    #可以用过实例对象引用
print(People.getCountry())    #可以通过类对象引用
"""
输出结果：
china
china
"""
```
类方法还有一个作用就是可以修改类属性：
```python
class People(object):
    country = 'china'

    #类方法，用classmethod来进行修饰
    @classmethod
    def getCountry(cls):
        return cls.country

    @classmethod
    def setCountry(cls,country):
        cls.country = country


p = People()
print(p.getCountry())    #可以用过实例对象引用
print(People.getCountry())    #可以通过类对象引用

p.setCountry('English')   

print(p.getCountry())    
print(People.getCountry())
"""
输出结果：
china
china
English
English
"""
```
# 静态方法
如果一个方法传递的参数不是和实例属性有关，那么就可以把它定义成静态方法，需要通过修饰器@staticmethod来进行修饰，静态方法不需要多定义参数。
```python
class People(object):
    country = 'china'

    @staticmethod
    def getCountry():
        return People.country


print (People.getCountry())
```
# 总结
从类方法和实例方法以及静态方法的定义形式就可以看出来，类方法的第一个参数是类对象cls，那么通过cls引用的必定是类对象的属性和方法；而实例方法的第一个参数是实例对象self，那么通过self引用的可能是类属性、也有可能是实例属性（这个需要具体分析），不过在存在相同名称的类属性和实例属性的情况下，实例属性优先级更高。静态方法中不需要额外定义参数，因此在静态方法中引用类属性的话，必须通过类对象来引用。
