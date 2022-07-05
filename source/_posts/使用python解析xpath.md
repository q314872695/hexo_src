---
title: 使用python解析xpath
date: 2022-07-05 15:26:26
tags: 爬虫
---

# xpath快速上手

1. 安装lxml库

```
pip install lxml
```

2. 使用lxml

	- `etree.parse()`：解析本地文件
	- `etree.HTML()`：解析服务器响应文件（比较常用）

```python
from lxml import etree
import requests
r = requests.get("https://www.baidu.com")
html_tree = etree.HTML(r.text)
result=html_tree.xpath("xpath路径")  # f
```



# xpath语法

1. 路径查询 

```python
//：查找所有子孙节点，不考虑层级关系 

/ ：找直接子节点 
```

2. 谓词查询 

```python
//div[@id] 

//div[@id="maincontent"] 
```

3. 属性查询 

```python
//@class 
```

4. 模糊查询 

```
//div[contains(@id, "he")] 

//div[starts‐with(@id, "he")] 
```

5. 内容查询 

```
//div/h1/text() 
```

6. 逻辑运算 

```
//div[@id="head" and @class="s_down"] 

//title | //price
```

