---
title: scrapy数据建模与请求
abbrlink: abe75b5d
date: 2022-07-09 17:53:36
tags:
 - scrapy
 - 爬虫
---




# scrapy数据建模与请求

**学习目标：**

1. 应用 在scrapy项目中进行建模
2. 应用 构造Request对象，并发送请求
3. 应用 利用meta参数在不同的解析函数中传递数据



## 1. 数据建模

> 通常在做项目的过程中，在items.py中进行数据建模

### 1.1 为什么建模

1. 定义item即提前规划好哪些字段需要抓，防止手误，因为定义好之后，在运行过程中，系统会自动检查
2. 配合注释一起可以清晰的知道要抓取哪些字段，没有定义的字段不能抓取，在目标字段少的时候可以使用字典代替
3. 使用scrapy的一些特定组件需要Item做支持，如scrapy的ImagesPipeline管道类，百度搜索了解更多

### 1.2 如何建模

在items.py文件中定义要提取的字段：

```python
class MyspiderItem(scrapy.Item): 
    name = scrapy.Field()   # 讲师的名字
    title = scrapy.Field()  # 讲师的职称
    desc = scrapy.Field()   # 讲师的介绍
```

### 1.3 如何使用模板类

模板类定义以后需要在爬虫中导入并且实例化，之后的使用方法和使用字典相同

job.py：

```python
from myspider.items import MyspiderItem   # 导入Item，注意路径
...
def parse(self, response)
    item = MyspiderItem() # 实例化后可直接使用

    item['name'] = node.xpath('./h3/text()').extract_first()
    item['title'] = node.xpath('./h4/text()').extract_first()
    item['desc'] = node.xpath('./p/text()').extract_first()

    print(item)
```

注意：

1. `from myspider.items import MyspiderItem`这一行代码中 注意item的正确导入路径，忽略pycharm标记的错误
2. python中的导入路径要诀：从哪里开始运行，就从哪里开始导入

### 1.4 开发流程总结

1. 创建项目
   `scrapy startproject 项目名`

2. 明确目标
   在items.py文件中进行建模

3. 创建爬虫

   3.1 创建爬虫

   ```shell
    scrapy genspider 爬虫名 允许的域
   ```

   3.2 完成爬虫

   ```
    修改start_urls
    检查修改allowed_domains
    编写解析方法
   ```

4. 保存数据
   在`pipelines.py`文件中定义对数据处理的管道
   在`settings.py`文件中注册启用管道

## 2. 翻页请求的思路

> 对于要提取如下图中所有页面上的数据该怎么办？

![img](https://halo-1257208482.piccd.myqcloud.com/202207121815843.png)

回顾requests模块是如何实现翻页请求的：

1. 找到下一页的URL地址
2. 调用`requests.get(url)`

scrapy实现翻页的思路：

1. 找到下一页的url地址
2. 构造url地址的请求对象，传递给引擎

## 3. 构造Request对象，并发送请求

### 3.1 实现方法

1. 确定url地址
2. 构造请求，`scrapy.Request(url,callback)`
   - `callback`：指定解析函数名称，表示该请求返回的响应使用哪一个函数进行解析
3. 把请求交给引擎：`yield scrapy.Request(url,callback)`

### 3.2 网易招聘爬虫

> 通过爬取网易招聘的页面的招聘信息,学习如何实现翻页请求
>
> 地址：https://hr.163.com/position/list.do

**思路分析：**

1. 获取首页的数据
2. 寻找下一页的地址，进行翻页，获取数据

**注意：**

1. 可以在`settings.py`中设置ROBOTS协议

```python
# False表示忽略网站的robots.txt协议，默认为True
ROBOTSTXT_OBEY = False
```

2. 可以在`settings.py`中设置User-Agent：

```python
# scrapy发送的每一个请求的默认UA都是设置的这个User-Agent
USER_AGENT = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36'
```

### 3.3 代码实现

在爬虫文件的parse方法中：

```python
......
    # 提取下一页的href
    next_url = response.xpath('//a[contains(text(),">")]/@href').extract_first()

    # 判断是否是最后一页
    if next_url != 'javascript:void(0)':

        # 构造完整url
        url = 'https://hr.163.com/position/list.do' + next_url

        # 构造scrapy.Request对象，并yield给引擎
        # 利用callback参数指定该Request对象之后获取的响应用哪个函数进行解析
        yield scrapy.Request(url, callback=self.parse)
......
```

### 3.4 scrapy.Request的更多参数

```python
scrapy.Request(url[,callback,method="GET",headers,body,cookies,meta,dont_filter=False])
```

**参数解释:**

1. 中括号里的参数为可选参数
2. `callback`：表示当前的url的响应交给哪个函数去处理
3. `meta`：实现数据在不同的解析函数中传递，meta默认带有部分数据，比如下载延迟，请求深度等
4. `dont_filter`:默认为False，会过滤请求的url地址，即请求过的url地址不会继续被请求，对需要重复请求的url地址可以把它设置为Ture，比如贴吧的翻页请求，页面的数据总是在变化;start_urls中的地址会被反复请求，否则程序不会启动
5. `method`：指定POST或GET请求
6. `headers`：接收一个字典，其中不包括cookies
7. `cookies`：接收一个字典，专门放置cookies
8. `body`：接收json字符串，为POST的数据，发送payload_post请求时使用（在下一章节中会介绍post请求）

## 4. meta参数的使用

> meta的作用：meta可以实现数据在不同的解析函数中的传递

在爬虫文件的parse方法中，提取详情页增加之前callback指定的parse_detail函数：

```python
def parse(self,response):
    ...
    yield scrapy.Request(detail_url, callback=self.parse_detail,meta={"item":item})
...

def parse_detail(self,response):
    #获取之前传入的item
    item = resposne.meta["item"]
```

**特别注意：**

1. meta参数是一个字典
2. meta字典中有一个固定的键`proxy`，表示代理ip，关于代理ip的使用我们将在scrapy的下载中间件的学习中进行介绍

------

## 小结

1. 完善并使用Item数据类：
   1. 在`items.py`中完善要爬取的字段
   2. 在爬虫文件中先导入Item
   3. 实力化Item对象后，像字典一样直接使用
2. 构造Request对象，并发送请求：
   1. 导入`scrapy.Request`类
   2. 在解析函数中提取url
   3. `yield scrapy.Request(url, callback=self.parse_detail, meta={})`
3. 利用meta参数在不同的解析函数中传递数据:
   1. 通过前一个解析函数 `yield scrapy.Request(url, callback=self.xxx, meta={})` 来传递meta
   2. 在self.xxx函数中 `response.meta.get('key', '')` 或 `response.meta['key']` 的方式取出传递的数据

------

### 参考代码

wangyi/spiders/job.py

```python
import scrapy


class JobSpider(scrapy.Spider):
    name = 'job'
    # 2.检查允许的域名
    allowed_domains = ['163.com']
    # 1 设置起始的url
    start_urls = ['https://hr.163.com/position/list.do']

    def parse(self, response):
        # 获取所有的职位节点列表
        node_list = response.xpath('//*[@class="position-tb"]/tbody/tr')
        # print(len(node_list))

        # 遍历所有的职位节点列表
        for num, node in enumerate(node_list):
            # 索引为值除2取余为0的才是含有数据的节点，通过判断进行筛选
            if num % 2 == 0:
                item = {}

                item['name'] = node.xpath('./td[1]/a/text()').extract_first()
                item['link'] = node.xpath('./td[1]/a/@href').extract_first()
                item['depart'] = node.xpath('./td[2]/text()').extract_first()
                item['category'] = node.xpath('./td[3]/text()').extract_first()
                item['type'] = node.xpath('./td[4]/text()').extract_first()
                item['address'] = node.xpath('./td[5]/text()').extract_first()
                item['num'] = node.xpath('./td[6]/text()').extract_first().strip()
                item['date'] = node.xpath('./td[7]/text()').extract_first()
                yield item

        # 翻页处理
        # 获取翻页url
        part_url = response.xpath('//a[contains(text(),">")]/@href').extract_first()

        # 判断是否为最后一页，如果不是最后一页则进行翻页操作
        if part_url != 'javascript:void(0)':
            # 拼接完整翻页url
            next_url = 'https://hr.163.com/position/list.do' + part_url
            yield scrapy.Request(url=next_url,callback=self.parse)
```

wangyi/items.py

```python
class WangyiItem(scrapy.Item):
    # define the fields for your item here like:

    name = scrapy.Field()
    link = scrapy.Field()
    depart = scrapy.Field()
    category = scrapy.Field()
    type = scrapy.Field()
    address = scrapy.Field()
    num = scrapy.Field()
    date = scrapy.Field()
```