---
title: python中的列表推导式
tags: python
abbrlink: 883e3be
date: 2019-08-20 15:24:00
---



所谓的列表推导式，就是指的轻量级循环创建列表。

- **基本使用方式**
```python
# 创建一个0-10的列表
a = [x for x in range(11)]
print(a)
"""
输出结果:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
"""
```
上面的列表推导式等价于下面，只是代码非常简化。
```python
a = []
for x in range(10):
	a.append(x)
```
- **在循环的过程中使用if**
```python
# 创建一个1-10之间偶数的列表
a = [x for x in range(11) if x%2==0]
"""
输出结果：
[0, 2, 4, 6, 8, 10]
"""
```
等价于
```python
a = []
for x in range(11):
	if x%2 == 0:
		a.append(x)
```
- **2个for循环**
```python
a = [(x,y) for x in range(3) for y in range(3)]
print(a)
"""
输出结果：
[(0, 0), (0, 1), (0, 2), (1, 0), (1, 1), (1, 2), (2, 0), (2, 1), (2, 2)]
"""
```
等价于
```python
a = []
for x in range(3):
	for y in range(3):
		a.append((x,y))
```
### 练习
- 生成一个[[1,2,3],[4,5,6]....]的列表最大值在100以内

首先考虑一下正常情况我们应该怎么写
```python
a = []
for x in range(1,100,3):
	a.append([x,x+1,x+2])
```
然后再把它转换成列表推导式
```python
a = [[x,x+1,x+2] for x in range(1,100,3)]
```
