---
title: 浅谈python中文件和文件夹的相关操作
tags: python
abbrlink: a9dd22f1
date: 2019-08-18 15:24:00
---



# 文件操作

## 文件的打开与关闭

 - 打开文件
使用**open**(文件名，访问方式)函数，可以打开一个已存在的文件，或者创建一个新的文件。
示例如下：
```python
f = open('test.txt') # 访问方式可以省略，默认以r(只读)的形式
f = open('test.txt', 'w')
f = open('test.txt', 'w', encoding="utf-8")
```
encoding默认时使用与操作系统一样的编码方式，window为gbk，linux为utf-8。在window中有的ide编码为utf-8，则在操作文件时需要额外设置encoding="utf-8"
|访问方式| 说明 |
|--|--|
| r | 以只读方式打开文本文件。文件的指针将会放在文件的开头。这是默认模式。 |
| w| 打开一文本个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| a| 打开一个文本文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。|
若要操作二进制文件（图片、视频等非文本文件），则在后面加**b**即可，例如**rb,wb,ab**。
若要想对文件又读又写，则在后面加+号，例如r+,w+,a+,rb+,wb+,ab+.
- 关闭文件
close() 方法用于关闭一个已打开的文件。
示例如下：
```python
f.close()
```
关闭后的文件不能再进行读写操作， 否则会触发 ValueError 错误。
## 文件的读写
- 写数据
使用write()可以完成向文件写入数据
当文件以文本文件打开时，write(str)传入的参数为str（字符串类型）
当文件以二进制文件打开时，write(bytes)传入的参数为bytes（二进制类型）
```python
# 以文本文件打开
f = open('test.txt', 'w', encoding="utf-8")
f.write('hello world, i am here!')
f.close()

# 以二进制文件打开
f = open('test.txt', 'wb')
f.write('hello world, i am here!'.encode("utf-8"))
f.close()
```
- 读数据（read）
使用read(num)可以从文件中读取数据，num表示要从文件中读取的数据的长度（单位是字节），如果没有传入num，那么就表示读取文件中所有的数据
示例如下：
```python
f = open('test.txt')

content = f.read(5)

print(content)

print("-"*30)

content = f.read()

print(content)

f.close()

"""
输出结果：
hello
------------------------------
 world, i am here!
"""
```
注意：如果读取了多次数据，那么后面读取的数据是从上次读完后的位置开始的
- 读数据（readline）
readline() 方法用于从文件读取整行，包括 "\n" 字符。如果指定了一个非负数的参数，则返回指定大小的字节数，包括 "\n" 字符。
文件test.txt中内容如下：

> 1:www.runoob.com 
> 2:www.runoob.com 
> 3:www.runoob.com 
> 4:www.runoob.com
> 5:www.runoob.com

示例如下：
```python
f = open('test.txt', 'r')

content = f.readline()

print(content)

print("-"*30)

content = f.readline()

print(content)

f.close()

"""
输出结果：
1:www.runoob.com

------------------------------
2:www.runoob.com
"""
```
- 读数据（readlines）
就像read没有参数时一样，readlines可以按照行的方式把整个文件中的内容进行一次性读取，并且返回的是一个列表，其中每一行的数据为一个元素。
示例如下：
```python
f = open('test.txt', 'r')

content = f.readlines()

print(content)

f.close()
"""
输出结果：
['1:www.runoob.com\n', '2:www.runoob.com\n', '3:www.runoob.com\n', '4:www.runoob.com\n', '5:www.runoob.com']
"""
```
## 文件的定位读写
- 获取当前读写的位置
在读写文件的过程中，如果想知道当前的位置，可以使用tell()来获取
文件test.txt中内容如下：

> 1:www.runoob.com 
> 2:www.runoob.com 
> 3:www.runoob.com 
> 4:www.runoob.com
> 5:www.runoob.com
```python
f = open("test.txt", "r")
str = f.read(3)
print("读取的数据是 : ", str)

# 查找当前位置
position = f.tell()
print("当前文件位置 : ", position)

str = f.read(3)
print("读取的数据是 : ", str)

# 查找当前位置
position = f.tell()
print("当前文件位置 : ", position)

f.close()
"""
读取的数据是 :  1:w
当前文件位置 :  3
读取的数据是 :  ww.
当前文件位置 :  6
"""
```
注意：文件位置从0开始记
- 定位到某个位置
如果在读写文件的过程中，需要从另外一个位置进行操作的话，可以使用seek()
seek(offset,from)有2个参数
 	- offset：偏移量
 	- from: 方向
 		- 0:表示文件开头
 		- 1:表示当前位置
 		- 2:表示文件末尾
```python
f = open("test.txt", "r")
str = f.readline()
print("读取的数据是 : ", str)

# 查找当前位置
position = f.tell()
print("当前文件位置 : ", position)

# 重新设置位置
f.seek(0, 0)

# 查找当前位置
position = f.tell()
print("当前文件位置 : ", position)

str = f.readline()
print("读取的数据是 : ", str)

f.close()
"""
输出结果：
读取的数据是 :  1:www.runoob.com

当前文件位置 :  18
当前文件位置 :  0
读取的数据是 :  1:www.runoob.com
"""
```
## 文件的重命名、删除
有时候需要对文件进行重命名、删除等操作时，要用到我们Python中的os模块，os 模块提供了非常丰富的方法用来处理文件和目录。
- 文件重命名
os模块中的rename()可以完成对文件的重命名操作。
语法：rename(需要修改的文件名，新的文件名)
```python
import os
# 将文件a重命名为文件b
os.rename("a.txt","b.txt")
```
- 删除文件
os模块中的remove()可以完成对文件的删除操作
语法：remove(待删除的文件名)
```python
import os
os.remove("a.txt")
```
# 文件夹操作
实际开发中，有时需要用程序的方式对文件夹进行一定的操作，比如创建、删除等就像对文件操作需要os模块一样，如果要操作文件夹，同样需要os模块。
- 创建文件夹
```python
import os
os.mkdir("文件夹")
```
- 获取当前目录
```python
import os
os.getcwd()
```
- 改变默认目录
```python
# 表示跳到上一级目录
os.chdir("../")
```
- 获取指定路径的目录列表
```python
import os
# 不传入参数时，返回当前的路径的列表
os.listdir(path)
```
- 删除文件夹
```python
import os
os.rmdir("文件夹")
```
