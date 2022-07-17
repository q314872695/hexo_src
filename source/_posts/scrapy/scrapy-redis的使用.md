---
title: scrapy-redis的使用
abbrlink: 5cc55a38
date: 2022-07-14 15:01:25
tags:
 - scrapy
 - 爬虫
---

# 前言

我电脑上现在已经安装的redis服务器，如果没有安装请先百度。

在实际开发中，我们一般先scrpay项目编写好，然后再把它改造成scrapy-redis

# scrapy-redis是什么

scrapy-redis是scrapy框架的基于redis的分布式组件，它在scrapy的基础上实现了更多，更强大的功能，具体体现在通过持久化请求队列和请求的指纹集合来实现：

- 断点续爬
- 分布式快速抓取

# 安装scrapy-redis

```python
pip install scrapy-redis
```

# 如何使用？

## 单机增量式爬虫

**复制以下内容到`setting.py`文件中**

```python
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
SCHEDULER_PERSIST = True
# SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderPriorityQueue" # 这个是默认的
# SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderQueue"
# SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderStack"

ITEM_PIPELINES = {
    'scrapy_redis.pipelines.RedisPipeline': 400,  # 把爬取的数据也存到redis中
}
#REDIS_HOST = 'localhost' # 默认
#REDIS_PORT = 6379
```

搞定，，就是这么简单，再运行你之前的scrapy爬虫即可实现断点续爬的功能，此时是一个单机增量式的爬虫，之前爬取过的url不会在请求了，每次运行只会爬取最新的内容。

## 分布式爬虫

### RedisSpider

在上面的基础上，把父类`scrapy.Spider`改成`RedisSpider`，然后添加属性`redis_key`，参考实例代码：

```python
from scrapy_redis.spiders import RedisSpider

class MySpider(RedisSpider):
    """Spider that reads urls from redis queue (myspider:start_urls)."""
    name = 'myspider_redis'

    # 注意redis-key的格式：
    redis_key = 'myspider:start_urls'

    # 可选：等效于allowd_domains()，__init__方法按规定格式写，使用时只需要修改super()里的类名参数即可
    def __init__(self, *args, **kwargs):
        # Dynamically define the allowed domains list.
        domain = kwargs.pop('domain', '')
        self.allowed_domains = filter(None, domain.split(','))

        # 修改这里的类名为当前类名
        super(MySpider, self).__init__(*args, **kwargs)

    def parse(self, response):
        return {
            'name': response.css('title::text').extract_first(),
            'url': response.url,
        }
```

**注意**：

RedisSpider类 不需要写`allowd_domains`和`start_urls`：

1. scrapy-redis将从在构造方法`__init__()`里动态定义爬虫爬取域范围，也可以选择直接写`allowd_domains`。
2. 必须指定redis_key，即启动爬虫的命令，参考格式：`redis_key = 'myspider:start_urls'`
3. 根据指定的格式，`start_urls`将在 Master端的 redis-cli 里 lpush 到 Redis数据库里，RedisSpider 将在数据库里获取start_urls。

**执行方式**：

1. 通过runspider方法执行爬虫的py文件（也可以分次执行多条），爬虫（们）将处于等待准备状态：
   
   `scrapy runspider myspider_redis.py`

2. 在Master端的redis-cli输入push指令，参考格式：
   
   `$redis > lpush myspider:start_urls http://www.dmoz.org/`

3. Slaver端爬虫获取到请求，开始爬取。

### RedisCrawlSpider

操作和上面的代码是一样的，只是继承的父类不同，从CrawlSpider->RedisCrawlSpider，实例代码如下：

```python
from scrapy.spiders import Rule
from scrapy.linkextractors import LinkExtractor

from scrapy_redis.spiders import RedisCrawlSpider


class MyCrawler(RedisCrawlSpider):
    """Spider that reads urls from redis queue (myspider:start_urls)."""
    name = 'mycrawler_redis'
    redis_key = 'mycrawler:start_urls'

    rules = (
        # follow all links
        Rule(LinkExtractor(), callback='parse_page', follow=True),
    )

    # __init__方法必须按规定写，使用时只需要修改super()里的类名参数即可
    def __init__(self, *args, **kwargs):
        # Dynamically define the allowed domains list.
        domain = kwargs.pop('domain', '')
        self.allowed_domains = filter(None, domain.split(','))

        # 修改这里的类名为当前类名
        super(MyCrawler, self).__init__(*args, **kwargs)

    def parse_page(self, response):
        return {
            'name': response.css('title::text').extract_first(),
            'url': response.url,
        }
```
