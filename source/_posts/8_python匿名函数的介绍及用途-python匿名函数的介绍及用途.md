---
title: python匿名函数的介绍及用途
date: 2019-08-17 15:24:00
tags: python
---

# 匿名函数

用lambda能够创建一个匿名函数，这种函数得名于省略了用def声明函数的标准步骤。
## 语法
```python
lambda [arg1 [,arg2,.....argn]]:expression
```
## 如何使用
1. 我们正常定义一个函数时是这样的
```python
def add(a,b):
	return a+b
```
2. 使用lambda定义匿名函数是这样的
```python
add = lambda a,b:a+b # 和上面函数功能一样
```
按正常的调用方式即可。lambda表达式能够接收任何数量的参数但只能返回一个表达式的值。
# 用途
### 1.就像上文所述能后够简化代码
### 2.匿名函数作为参数传递
1. 作为自定义函数的参数传递
```python
def test(a, b, func):
    result = func(a, b)
    print(result)


func_new = input("请输入一个匿名函数：")
# eval()将字符串str当成有效的表达式来求值并返回计算结果。
func_new = eval(func_new)

test(11, 22, func_new)

"""
输出结果：
请输入一个匿名函数：lambda a,b:a+b
33
"""
```
2. 作为内置函数的参数传递
例如：将列表中的字典按照指定的关键字进行排序
```python
stus = [
    {"name":"zhangsan", "age":18}, 
    {"name":"lisi", "age":19}, 
    {"name":"wangwu", "age":17}
]
stus.sort(key = lambda x:x['age'])

for stu in stus:
	print(stu)

"""
输出结果：
{'name': 'wangwu', 'age': 17}
{'name': 'zhangsan', 'age': 18}
{'name': 'lisi', 'age': 19}
"""
```
