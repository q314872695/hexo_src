---
title: python对常见数据类型的遍历
date: 2019-08-12 15:24:00
tags: python
---


**本文将通过for ... in ...的语法结构，遍历字符串、列表、元组、字典等数据结构。**

# 字符串遍历
```python
>>> a_str = "hello itcast"
>>> for char in a_str:
...     print(char,end=' ')
...
h e l l o   i t c a s t
```
# 列表遍历
```python
>>> a_list = [1, 2, 3, 4, 5]
>>> for num in a_list:
...     print(num,end=' ')
...
1 2 3 4 5
```
# 元组遍历

```python
>>> a_turple = (1, 2, 3, 4, 5)
>>> for num in a_turple:
...     print(num,end=" ")
1 2 3 4 5
```
# 字典遍历

 1. **遍历字典的key（键）**


```python
>>> a_dict = {"name":"lxy","sex":"男","age":18}
>>> for key in a_dict.keys():
	print(key)

	
name
sex
age
```

 2. **遍历字典的value（值）**
```python
>>> a_dict = {"name":"lxy","sex":"男","age":18}
>>> for value in a_dict.values():
	print(value)


lxy
男
18
>>> 
```
3. **遍历字典的项（元素）**

```python
>>> a_dict = {"name":"lxy","sex":"男","age":18}
>>> for key,value in a_dict.items():
	print("key=%s,value=%s"%(key,value))

	
key=name,value=lxy
key=sex,value=男
key=age,value=18
```
# 重点：带下标索引的遍历

 - 正常情况我们是这样的


```python
>>> chars = ['a', 'b', 'c', 'd']
>>> i = 0
>>> for chr in chars:
...     print("%d %s"%(i, chr))
...     i += 1
...
0 a
1 b
2 c
3 d
```

 - 升级版使用**enumerate()**
	 - 介绍：enumerate()函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。
	 - 语法：enumerate(sequence, [start=0])
	 - 参数：
	 	- sequence -- 一个序列、迭代器或其他支持迭代对象。
	 	- start -- 下标起始位置。 
	 - 返回值：返回 tuple(元组) 对象。

```python
>>> chars = ['a', 'b', 'c', 'd']
>>> for i, chr in enumerate(chars):
	print(i,chr)

	
0 a
1 b
2 c
3 d
>>> 
```
