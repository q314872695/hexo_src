---
title: Selenium入门笔记
tags: 爬虫
abbrlink: d86439ce
date: 2022-07-06 14:38:16
---


# 什么是selenium

1. Selenium是一个用于Web应用程序测试的工具。 
2. Selenium 测试直接运行在浏览器中，就像真正的用户在操作一样。 
3. 支持通过各种driver（FirfoxDriver，IternetExplorerDriver，OperaDriver，ChromeDriver）驱动真实浏览器完成测试。 
4. selenium也是支持无界面浏览器操作的。 

# 为什么使用selenium

模拟浏览器功能，自动执行网页中的js代码，实现动态加载

# 如何安装selenium？ 

1. 根据电脑上使用的浏览器下载对应的浏览器驱动， [安装浏览器驱动 ](https://www.selenium.dev/zh-cn/documentation/webdriver/getting_started/install_drivers/)
2. 把下载的驱动的`.exe`文件添加到环境变量的path路径中。
	
	1. 例如我电脑使用的浏览器是win10自带的Edge浏览器，下载浏览器驱动得到一个压缩包，解压后得到`msedgedriver.exe`
	
	![image-20220706145832890](https://halo-1257208482.image.myqcloud.com/202207080922592.png!webp)
	
	2. 添加到环境变量中
	
	![image-20220706145940482](https://halo-1257208482.image.myqcloud.com/202207080922594.png!webp)
	
	3. 在 `cmd` 中输入 `msedgedriver` ，若出现如下页面则设置成功。
   
	![image-20220706150220464](https://halo-1257208482.image.myqcloud.com/202207080922595.png!webp)
	
3. 安装selenium `pip install selenium`

# selenium的元素定位？ 

**元素定位：**自动化要做的就是模拟鼠标和键盘来操作来操作这些元素，点击、输入等等。操作这些元素前首先 

要找到它们，WebDriver提供很多定位元素的方法 

**方法：**

- `find_element(by=By.ID, value=None)`：返回一个元素
- `find_elements(by=By.ID, value=None)`：返回一个列表

其中by参数是根据什么来查找这个元素，默认是根据`id`来查找，同时也提供如下类型：

```python
class By:
    """
    Set of supported locator strategies.
    """
    ID = "id"
    XPATH = "xpath"
    LINK_TEXT = "link text"
    PARTIAL_LINK_TEXT = "partial link text"
    NAME = "name"
    TAG_NAME = "tag name"
    CLASS_NAME = "class name"
    CSS_SELECTOR = "css selector"
```

**注意：**selenium提供的查找方法，只能查找到元素信息，具体的属性信息无法查找，需要使用下文提到的方法。

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

browser = webdriver.Edge()
browser.get("https://www.baidu.com")
# 以使用xpath为例
# img = browser.find_element(by=By.XPATH, value="//img/@src") 错误写法，无法获取元素属性
img = browser.find_element(by=By.XPATH, value="//img") # 正确写法
src = img.get_attribute("src") # 获取元素属性
```



# 访问元素信息

必须是`find_element`或者`find_elements`返回的对象才有如下方法

- `get_attribute('属性名称') `：获取元素属性 
- `text ` ：获取元素文本 
- `tag_name` ：获取标签名 

# 交互

```python
点击:click() 
输入:send_keys() 
后退操作:browser.back() 
前进操作:browser.forword() 
模拟JS滚动: 
    js='document.documentElement.scrollTop=100000'
    browser.execute_script(js) 执行js代码 
获取网页代码：page_source
退出：browser.quit()
```



# Edge浏览器无头模式

由于无头浏览器不进行css和gui渲染，运行效率要比真实的浏览器要快很多 

```python
from selenium import webdriver
from selenium.webdriver.edge.options import Options

# 配置参数，其他浏览器可能稍有不同
options = Options()
options.add_argument("headless")
driver = webdriver.Edge(options=options)

...
```

# 代理

```python
from selenium import webdriver
from selenium.webdriver.edge.options import Options

# 设置代理
PROXY = "112.6.117.178:8085"
webdriver.DesiredCapabilities.EDGE['proxy'] = {
    "httpProxy": PROXY,
    "ftpProxy": PROXY,
    "sslProxy": PROXY,
    "proxyType": "MANUAL",

}
....

```



# 更多信息参考官方文档

[官方文档](https://www.selenium.dev/zh-cn/documentation/)





.
