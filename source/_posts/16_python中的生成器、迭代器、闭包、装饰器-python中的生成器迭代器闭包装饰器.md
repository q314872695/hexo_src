---
title: python中的生成器、迭代器、闭包、装饰器
date: 2019-08-22 15:24:00
tags: python
---



# 迭代器

迭代是访问集合元素的一种方式。迭代器是一个可以记住遍历的位置的对象。迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退。
## 可迭代对象
以直接作用于 for 循环的数据类型有以下几种：

- 一类是集合数据类型，如 list 、 tuple 、 dict 、 set 、 str 等；

- 一类是 generator ，包括生成器和带 yield 的generator function。

这些可以直接作用于 for 循环的对象统称为可迭代对象： Iterable 。

## 判断是否可以迭代
可以使用 isinstance() 判断一个对象是否是 Iterable 对象：
```python
from collections import Iterable
isinstance([],Iterable)
# True
isinstance({},Iterable)
# True
isinstance(123,Iterable)
# False
isinstance((x for x in range(10)),Iterable)
# True
```
## 什么是迭代器
**可以被next()函数调用并不断返回下一个值的对象称为迭代器：Iterator。**
可以使用 isinstance() 判断一个对象是否是 Iterator 对象：
```python
from collections import Iterator
isinstance([],Iterator)
False
isinstance({},Iterator)
False
isinstance((x for x in range(10)),Iterator) # 
True
```
生成器都是迭代器。

## iter()函数
虽然list 、 tuple 、 dict 、 set 、 str 等是可迭代对象，但他们不是迭代器。可以通过iter()函数把可迭代对象编程迭代器。
```python
isinstance(iter([]),Iterator)
# True
isinstance(iter({}),Iterator)
# True
isinstance(iter("asdf"),Iterator)
# True
```
## 总结：
- 凡是可作用于 for 循环的对象都是 Iterable 类型。
- 凡是可作用于 next() 函数的对象都是 Iterator 类型。
- 集合数据类型如 list 、 dict 、 str 等是 Iterable 但不是 Iterator ，不过可以通过 iter() 函数获得一个 Iterator 对象。
# 生成器
## 什么是生成器
我们可以通过列表生成式来创建一个列表，但是收到内存的限制，列表的容量肯定是有限的。而且，创建一个包含100万个元素的列表，不仅占用很大的存储空间，如果我们仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了。所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。**在Python中，这种一边循环一边计算的机制，称为生成器：generator。**

## 修改列表推导式创建生成器的方法
最简单的方法是把列表生成式中的 [ ] 改成 ( ) 就好了。
```python
a = [x for x in range(10)]
print(a) # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
b = (x for x in range(10))
print(b) # <generator object <genexpr> at 0x03387DB0>
```
### 如何遍历生成器
我们发现生成器不是能直接打印出来的，我们可以通过next()函数来获得生成器的下一个返回值。
生成器保存的是算法，每次调用 next(G) ，就计算出 G 的下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出 StopIteration 的异常。
**使用next()  或者__next __():**
```python
print(next(b))
# 0
print(next(b))
# 1
print(next(b))
# 2
print(next(b))
# 3
print(next(b))
# 4
print(next(b))
# 5
print(b.__next__())
# 6
print(b.__next__())
# 7
print(b.__next__())
# 8
print(b.__next__())
# 9
print(b.__next__())
# Traceback (most recent call last):
#   File "<input>", line 2, in <module>
# StopIteration
```
那么有什么简单的方法呢？因为生成器是可迭代对象，也可以**使用for循环来遍历它**，并且不需要关心 StopIteration 异常。
```python
b = (x for x in range(10))
for x in b:
    print(x)
    
# 0
# 1
# 2
# 3
#...
```

## 函数中使用yield创建生成器的方法
如果如果生成器推算的算法比较复杂，用类似列表生成式的 for 循环无法实现的时候，还可以用函数来实现。把你要返回的值前面加**yield** 即可。

使用函数实现上面代码：
```python
def fn():
	for x in range(3):
		yield x
# 遍历函数实现的生成器
f = fn()
print(next(f))
# 0
print(next(f))
# 1
print(next(f))
# 2
print(next(f))
# Traceback (most recent call last):
#   File "<input>", line 1, in <module>
# StopIteration
```

使用生成器实现斐波拉契数列：
```python
def fib(count):
	n = 0
	a,b = 0,1
	while n < count:
		yield b
		a,b = b,a+b
		n += 1
	return "done"
f = fib(5)
print(next(f))
# 1
print(next(f))
# 1
print(next(f))
# 2
print(next(f))
# 3
print(next(f))
# 5
print(next(f))
# Traceback (most recent call last):
#   File "<input>", line 1, in <module>
# StopIteration: done
```
### yield执行流程
- **当执行next(f)时，函数开始执行到yield，yield 右边的变量x作为next()的返回值被返回，此时函数保存当前的运行状态，并暂停执行。**
- **再次调用next(f)时，函数从上次暂停的位置开始继续执行，再次遇到yield时重复上面的操作**
- **直到生成器遍历结束**

我们在循环过程中不断调用 yield ，就会不断中断。当然要给循环设置一个条件来退出循环，不然就会产生一个无限数列出来。同样的，把函数改成generator后，我们基本上从来不会用 next() 来获取下一个返回值，而是直接使用 for 循环来迭代：
```python
for x in fib(5):
	print(x)
# 1
# 1
# 2
# 3
# 5
```
但是用for循环调用generator时，发现拿不到generator的return语句的返回值。如果想要拿到返回值，必须捕获StopIteration错误，返回值包含在StopIteration的value中：
```python
f = fib(5)
while True:
	try:
		print(next(f))
	except StopIteration as e:
		print("生成器返回值:%s"%e.value)
		break
# 1
# 1
# 2
# 3
# 5
# 生成器返回值:done
```

## send方法
```python
def gen():
	i = 0
	while i<3:
		temp = yield i
		print(temp)
		i+=1
g = gen()
print(g.__next__())
# 0
print(g.send(None))
# None
# 1
print(next(g))
# None
# 2
print(g.send("哈哈")
# 哈哈
# Traceback (most recent call last):
#   File "<input>", line 1, in <module>
# StopIteration
```
- **上面代码可以看出next()、__next __()、send(None)是等价的并没有什么区别。**
- **send()其实是比他们更高级的，在之前的代码中yield i是没有返回值的即输出为None。**
- **如果修改send()的形参，那么yield i 的返回值就是括号中的形参，在上面的代码中g.send("哈哈")相当于temp = "哈哈"，并且g.send("哈哈")的返回值就是变量i。**
- **使用send时要注意，第一次调用生成器对象时，send()不能传参数否则会报错，第一次必须是send(None)，或者第一次调用next()、__next __()也可以。**

# 闭包
## 什么是闭包
在函数内部再定义一个函数，并且这个函数用到了外边函数的变量，那么将这个函数以及用到的一些变量称之为闭包
```python
def test(number):
	def test_in(number_in):
		print("test_in函数 的number_in=%s"%number_in)
		return number_in+number
	# 返回test_in函数的引用
	return test_in
ret = test(20)
print(ret(100)) # 相当于直接调用test_in函数，并给它传值100
# test_in函数 的number_in=100
# 120
print(ret(200))
# test_in函数 的number_in=200
# 220
```
## 闭包的一个例子
在数学中，一次函数：y=kx+b，在一条确定的直线中，它的k、b是不变的。求y时，根据确定的k、b、x来求出。
```python
def line_conf(k, b):
    def line(x):
        return k*x + b
    return line

line1 = line_conf(1, 1)
line2 = line_conf(4, 5)
print(line1(5))
print(line2(5))
# 6
# 25
```
如果没有闭包，我们需要每次创建直线函数的时候同时说明a,b,x。这样，我们就需要更多的参数传递，也减少了代码的可移植性。

# 装饰器
## 什么是装饰器
装饰器就是对一个函数进行装饰，给这个函数增加额外的功能。
```python
def logging(func):
    def wrap():
        print("正在打印日志！")
        func()
    return wrap

@logging # 该装饰器为函数增加了打印日志的额外功能，并且之前函数内部代码不会改变。
def login():
	print("张三正在登陆。")
login()
# 正在打印日志！
# 张三正在登陆。
```
## 两个装饰器
```python
def makeBold(fn1):
    def wrapped():
        print("----1----")
        return "<b>"+fn1()+"</b>"
    return wrapped

def makeItalic(fn2):
    def wrapped():
        print("----2----")
        return "<i>"+fn2()+"</i>"
    return wrapped

@makeBold
@makeItalic
def f1():
    print("----3----")
    return "hello world"

ret = f1() # 此时f1并不是f1函数，它是makeBold装饰器返回的wrapped函数的引用。
print(ret)
"""
输出结果：
----1----
----2----
----3----
<b><i>hello world</i></b>
"""
```
调用流程：
1. 把函数f1的引用传入装饰器makeItalic中的变量fn2，此时fn2指向f1函数。
2. 把装饰器makeItalic中wrapped函数的引用传入装饰器makeBold的变量fn1，此时fn1指向装饰器makeItalic中的wrapped函数。
3. ret = f1()表是执行f1所指向的函数，并返回给ret。

## 装饰器带参数
一般情况下装饰器内部函数的参数都是不定长参数，保证通用性，确保装饰任何函数时都不会出错。
```python
def logging(func):
    def wrap(*args,**kwargs):
        print("正在打印日志！")
        func(*args,**kwargs)
    return wrap

@logging # 该装饰器为函数增加了打印日志的额外功能，并且之前函数内部代码不会改变。
def login(name,dic):
	print("%s正在登陆。"%name)
	print(dic)
login("李四",{"sex":"男"})
# 正在打印日志！
# 李四正在登陆。
# {'sex': '男'}
```

## 类装饰器
```python
class Test(object):
    def __init__(self, func):
        print("---初始化---")
        print("func name is %s"%func.__name__)
        self.__func = func
    def __call__(self):
        print("---装饰器中的功能---")
        self.__func()
@Test
def test():
    print("----test---")
test()
"""
输出结果：
---初始化---
func name is test
---装饰器中的功能---
----test---
"""
```
说明：
- 当用Test来装作装饰器对test函数进行装饰的时候，首先会创建Test的实例对象并且会把test这个函数名当做参数传递到__init__方法中即在__init__方法中的func变量指向了test函数体。
- test函数相当于指向了用Test创建出来的实例对象。
- 当在使用test()进行调用时，就相当于让这个对象()，因此会调用这个对象的__call__方法。
- 为了能够在__call__方法中调用原来test指向的函数体，所以在__init __方法中就需要一个实例属性来保存这个函数体的引用所以才有了self.__func = func这句代码，从而在调用__call __方法中能够调用到test之前的函数体。
