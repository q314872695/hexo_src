---
title: import-xxx和from-xxx-import-xx中修改被导入模块里面的变量的坑
tags: python
abbrlink: d46a9e4f
date: 2019-08-29 15:24:00
---



现在有如下几个模块：
**A.py**
功能：定义全局变量，供其他模块使用

```python
name = "张三"
lists = [1, 2, 3, 4, 5]
```
**B.py**
功能：打印A.py中的变量
```python
from A import name,lists
def test():
    print("B:",name)
    print("B:",lists)
```
**main.py**
```python
from A import name,lists
from B import test

if __name__ == '__main__':
    print("修改前-main:",name)
    name = "李四"
    print("修改后-main:", name)
    print("main:",lists)
    
    lists.append(100)
    # B模块中test的功能是打印A模块的name、lists
    test()
 
"""
修改前-main: 张三
修改后-main: 李四
main: [1, 2, 3, 4, 5]
B: 张三
B: [1, 2, 3, 4, 5, 100]
"""
```
**运行后我们发现：**
- A模块中的name被修改了，但在B模块中打印的还是原来的值。
- A模块中的lists添加了新的元素，显示正常。

**原因：**
- 使用from A import name 是在在当前模块创建一个name变量，该变量指向A模块中name变量所指向的值,即张三，修改name的值其实是修改name的引用，而字符串为不可变类型，name = "李四"，是在内存中创建了字符串李四，然后name重新指向李四，所以之前的张三并没有改变。
- 而lists它是一个列表，是可变类型，所以在列表内添加新元素是可行的，但是lists是不能被重新赋值的，重新赋值时它的引用就会改变。


**解决办法：**
- 把定义全局变量的A.py使用import 导入。

就像这样：
**B.py**
```python
import A
def test():
    print("B:",A.name)
    print("B:",A.lists)
```
**main.py**
```python
import A
from B import test

if __name__ == '__main__':
    print("修改前-main:",A.name)
    name = "李四"
    print("修改后-main:", A.name)
    print("main:",A.lists)

    A.lists.append(100)
    # B模块中test的功能是打印A模块的name、lists
    test()
"""
修改前-main: 张三
修改后-main: 张三
main: [1, 2, 3, 4, 5]
B: 张三
B: [1, 2, 3, 4, 5, 100]
"""
```


