---
title: scrapy管道的使用
abbrlink: b819e60d
date: 2022-07-10 18:34:59
tags:
 - scrapy
 - 爬虫
---



# scrapy管道的使用

**学习目标**：

1. 掌握 scrapy管道(pipelines.py)的使用

------

> 之前我们在scrapy入门使用一节中学习了管道的基本使用，接下来我们深入的学习scrapy管道的使用

## 1. pipeline中常用的方法：

1. `process_item(self,item,spider):`
   - 管道类中必须有的函数
   - 实现对item数据的处理
   - 必须return item
2. `open_spider(self, spider): `在爬虫开启的时候仅执行一次
3. `close_spider(self, spider):` 在爬虫关闭的时候仅执行一次

## 2. 管道文件的修改

> 继续完善wangyi爬虫，在pipelines.py代码中完善

```python
import json
from pymongo import MongoClient

class WangyiFilePipeline(object):
    def open_spider(self, spider):  # 在爬虫开启的时候仅执行一次
        if spider.name == 'itcast':
            self.f = open('json.txt', 'a', encoding='utf-8')

    def close_spider(self, spider):  # 在爬虫关闭的时候仅执行一次
        if spider.name == 'itcast':
            self.f.close()

    def process_item(self, item, spider):
        if spider.name == 'itcast':
            self.f.write(json.dumps(dict(item), ensure_ascii=False, indent=2) + ',\n')
        # 不return的情况下，另一个权重较低的pipeline将不会获得item
        return item  

class WangyiMongoPipeline(object):
    def open_spider(self, spider):  # 在爬虫开启的时候仅执行一次
        if spider.name == 'itcast':
        # 也可以使用isinstanc函数来区分爬虫类:
            con = MongoClient(host='127.0.0.1', port=27017) # 实例化mongoclient
            self.collection = con.itcast.teachers # 创建数据库名为itcast,集合名为teachers的集合操作对象

    def process_item(self, item, spider):
        if spider.name == 'itcast':
            self.collection.insert(item) 
            # 此时item对象必须是一个字典,再插入
            # 如果此时item是BaseItem则需要先转换为字典：dict(BaseItem)
        # 不return的情况下，另一个权重较低的pipeline将不会获得item
        return item
```

## 3. 开启管道

在settings.py设置开启pipeline

```python
......
ITEM_PIPELINES = {
    'myspider.pipelines.ItcastFilePipeline': 400, # 400表示权重
    'myspider.pipelines.ItcastMongoPipeline': 500, # 权重值越小，越优先执行！
}
......
```

**别忘了开启mongodb数据库 `sudo service mongodb start`** **并在mongodb数据库中查看 `mongo`**

**思考：在settings中能够开启多个管道，为什么需要开启多个？**

1. 不同的pipeline可以处理不同爬虫的数据，通过spider.name属性来区分
2. 不同的pipeline能够对一个或多个爬虫进行不同的数据处理的操作，比如一个进行数据清洗，一个进行数据的保存
3. 同一个管道类也可以处理不同爬虫的数据，通过spider.name属性来区分

## 4. pipeline使用注意点

1. 使用之前需要在settings中开启
2. pipeline在setting中键表示位置(即pipeline在项目中的位置可以自定义)，值表示距离引擎的远近，越近数据会越先经过：**权重值小的优先执行**
3. 有多个pipeline的时候，process_item的方法必须return item,否则后一个pipeline取到的数据为None值
4. pipeline中`process_item()`的方法必须有，否则item没有办法接受和处理
5. `process_item`()方法接受item和spider，其中spider表示当前传递item过来的spider
6. `open_spider(spider)` :能够在爬虫开启的时候执行一次
7. `close_spider(spider)` :能够在爬虫关闭的时候执行一次
8. 上述俩个方法经常用于爬虫和数据库的交互，在爬虫开启的时候建立和数据库的连接，在爬虫关闭的时候断开和数据库的连接


## 小结

- 管道能够实现数据的清洗和保存，能够定义多个管道实现不同的功能，其中有个三个方法
  - `process_item(self,item,spider):`实现对item数据的处理
  - `open_spider(self, spider): `在爬虫开启的时候仅执行一次
  - `close_spider(self, spider): `在爬虫关闭的时候仅执行一次
