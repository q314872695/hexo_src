---
title: request+xpath爬取免费代理打造自己的ip代理池
tags: 爬虫
abbrlink: 9dd36ef2
date: 2022-07-03 21:27:37
---

> 免费代理对于咱们羊毛党来说是个好动西，但是免费的有好多的都不能用，况且还得一个一个试，累都累死了，于是写了这么一个小案例，用于减轻劳动。

# 爬虫代码

本案例是爬取的快代理的免费匿名代理，并自动检测代理是否可用，把可用的代理以json文件保存起来

```python
from lxml import etree
import requests
import json

"""
打造自己的ip代理池
"""


def get_ip_pool():
    # 用集合去重
    ip_pool = set()
    # 爬取前5页的免费代理
    page = 5
    for i in range(1, page + 1):
        url = 'https://free.kuaidaili.com/free/inha/' + str(i)
        r = requests.get(url)
        tree = etree.HTML(r.text)
        ip_list = tree.xpath(
            "/html/body/div[@class='body']/div[@id='content']/div[@class='con-body']/div[2]/div[@id='list']/table["
            "@class='table table-bordered table-striped']/tbody/tr")
        for i in ip_list:
            ip = i.xpath("./td[1]/text()")[0]
            port = i.xpath("./td[2]/text()")[0]
            print("{}:{}".format(ip, port))
            ip_pool.add(ip + ":" + port)
    return ip_pool


# 检测代理ip是否可用
def get_active_ip(pool):
    ips = []
    for item in pool:
        try:
            r = requests.get("http://4.ipw.cn/", proxies={"http": item}, timeout=1)
            if r.status_code == 200:
                print("{}可用---------------------".format(item))
                ips.append(item)
        except Exception as e:
            print("{}不可用".format(item))
    return ips


pool = get_ip_pool()
ip_pool = get_active_ip(pool)
# 持久化，把有效的ip保存起来
with open("ip.json", "w", encoding="utf-8") as f:
    f.write(json.dumps(ip_pool))
    
# ip.json文件中的内容
[
  "112.6.117.178:8085",
  "223.96.90.216:8085",
  "106.55.15.244:8889",
  "58.20.184.187:9091",
  "120.194.55.139:6969",
  "122.9.101.6:8888"
]

```

# 使用代理

随机从中选择一个代理使用，具体发送http请求的代码自己实现吧

```python
import json
import random
f = open("ip.json", "r", encoding="utf-8")
ip_pool = json.loads(f.read())
# 随机选择一个并打印出来
print(random.choice(ip_pool))
f.close()

```

