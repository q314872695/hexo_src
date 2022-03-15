---
title: python中的元类介绍
tags: python
abbrlink: 34b6116e
date: 2019-08-21 15:24:00
---



# 类也是对象

在大多数编程语言中，类就是一组用来描述如何生成一个对象的代码段，在python中也是成立的。
```python
class ObjectCreator:
	pass
	
my_object = ObjectCreator()
print(my_object)
"""
输出结果：
<__main__.ObjectCreator object at 0x037DACD0>
"""
```
但是，python的类不止于此，类同样也是一种对象。
```python
class ObjectCreator:
	pass
```
上面的代码段将在内存中创建一个对象，名字就叫做ObjectCreator。这个对象（类对象ObjectCreator）拥有创建对象（实例对象）的能力，但它本质上仍然还是一个对象，于是你就可以对它做如下的操作：
- 给它复制一个变量
- 拷贝它
- 给它增加属性
- 将它作为函数参数传递
示例代码：
```python
class ObjectCreator:
    pass
# 把它赋值给一个变量
a = ObjectCreator
print(a) # <class '__main__.ObjectCreator'>
# 作为函数参数传递
def echo(o):
    print(o)
    
echo(ObjectCreator) # <class '__main__.ObjectCreator'>
```
# 动态的创建类
因为类也是对象，所以可以在运行时动态的创建它们，使用class关键字即可。
```python
def choose_class(name):
    if name == 'foo':
        class Foo(object):
            pass
        return Foo     # 返回的是类，不是类的实例
    else:
        class Bar(object):
            pass
        return Bar
MyClass = choose_class("foo")
print(MyClass) # 打印类对象
print(MyClass()) # 打印实例对象
# 输出结果
# <class '__main__.choose_class.<locals>.Foo'>
# <__main__.choose_class.<locals>.Foo object at 0x0368CFD0>
```
# 使用type创建类
我们知道通过type()可以知道这个对象的类型是什么，他还有一个完全不同的功能，动态的创建类。
type可以接受一个类的描述作为参数，然后返回一个类。
语法：
**type(类名，由父类名称构成的元组（针对继承的情况可以为空），包含属性的字典)**
```python
MyClass = type("MyClass",(),{})
print(MyClass)
# 输出结果：
# <class '__main__.MyClass'>
```
## 使用type创建带属性的类
type 接受一个字典来为类定义属性，如下所示：
```python
Foo = type("Foo",(),{'bar':True})
```
等价于
```python
class Foo:
	bar = True
```
## 使用type创建继承的子类
接着上面的代码，我们已经创建了一个Foo类，现在来创建一个它的子类。
```python
FooChild = type("FooChild",(Foo,),{})
print(FooChild.bar) # # bar属性是由Foo继承而来
# 输出结果：
True
```
注意：
- type的第二个参数，元组中是父类的名字，不是字符串。
- 添加的属性是类属性，不是实例属性。

## 使用type创建带有方法的类
最终你会希望为你的类增加方法。只需要定义一个有着恰当签名的函数并将其作为属性赋值就可以了。
### 添加实例方法
```python
def test_f(self):
	print("添加的实例方法")
Foo = type("Foo",(),{"test_f":test_f})
f = Foo()
f.test_f()
# 输出结果：
# 添加的实例方法
```

### 添加静态方法
```python
@staticmethod
def test_static():
	print("添加的静态方法")
Foo = type("Foo",(),{"test_static":test_static})
Foo.test_static()
Foo.test_static()
# 输出结果：
# 添加的静态方法
```

### 添加类方法
```python
@classmethod
def test_class(cls):
	print("添加的类方法")
Foo = type("Foo",(),{"test_class":test_class})
Foo.test_class()
# 输出的结果：
#添加的类方法
```

# 什么是元类
元类就是用来创建类的“东西”。元类就是就是用来创建类对象的，元类就是类的类。
可以这样理解：

```python
MyClass = 	MetaClass() # 使用元类创建类对象
MyObject = MyClass() # 使用类对象创建实例对象
```

type函数其实就是元类。type就是在Python在背后创建所有类的元类，可以通过__class __属性来查看,__class __的**功能是查看对象所在的类**，它可以嵌套使用。
```python
class A:
    pass
print(A.__class__)
a = A()
print(a.__class__)
print(a.__class__.__class__)
# 输出结果：
# <class 'type'>
# <class '__main__.A'>
# <class 'type'>
```
可以看出，最后对象的类都是type元类。
**Python中所有的东西，注意，我是指所有的东西——都是对象。这包括整数、字符串、函数以及类。它们全部都是对象，而且它们都是从一个类创建而来，这个类就是type。**
整数：
```python
age = 18
print(age.__class__)
print(age.__class__.__class__)
# 输出结果：
# <class 'int'>
# <class 'type'>
```
字符串：
```python
name = "张三"
print(name .__class__)
print(name .__class__.__class__)
# 输出结果：
# <class 'str'>
# <class 'type'>
```
函数：
```python
def f():
	pass
print(f.__class__)
print(f.__class__.__class__)
# 输出结果：
# <class 'function'>
# <class 'type'>
```

# 自定义元类
首先的了解一下metaclass属性，用它来指定一个元类，python会在定义的类中寻找metaclass属性，如果没找到，就到它的父类找以此类推。如果找到了，python就会用它来创建类对象，如果实在没有找到就会用内建的type来创建这个类。
metaclass中可以放type或者任何使用到type或者子类化type的东东都可以。
自定义类的主要目的：
- 拦截类的创建
- 修改类
## 使用函数实现一个自定义的元类
功能：把不是__开头的类属性名字变为大写
```python
def upper_attr(future_class_name: str,future_class_parents: tuple,future_class_attr: dict):
	newAttr = {}
	for key,value in future_class_attr.items():
		if not key.startswith("__"):
			newAttr[key.upper()] = value
	return type(future_class_name,future_class_parents,newAttr)
class Foo(metaclass=upper_attr):
	name = "张三"
	age = 18

hasattr(Foo,"name") # 判断是否有该类属性	False
hasattr(Foo,"NAME") # True
hasattr(Foo,"age") # False
hasattr(Foo,"AGE") # True
```

## 继承type实现一个自定义元类
功能：同上
```python
class MyMetaClass(type):
    def __new__(cls, class_name: str, class_parents: tuple, class_attr: dict):
        newAttr = {}
        for key, value in class_attr.items():
            if not key.startswith("__"):
                newAttr[key.upper()] = value

        # 方法1：通过'type'来做类对象的创建
        # return type(class_name, class_parents, newAttr)
    
        # 方法2：复用type.__new__方法
        # 这就是基本的OOP编程，没什么魔法
        # return type.__new__(cls, class_name, class_parents, newAttr)
    
        # 方法3：使用super方法
        return super(MyMetaClass, cls).__new__(cls, class_name, class_parents, newAttr)


class Foo(metaclass=MyMetaClass):
    name = "张三"
    age = 18

hasattr(Foo,"name") # 判断是否有该类属性	False
hasattr(Foo,"NAME") # True
hasattr(Foo,"age") # False
hasattr(Foo,"AGE") # True
```
效果和上面是一样的。
