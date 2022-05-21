---
title: matplotlib学习笔记
tags: matplotlib
abbrlink: 643c4929
date: 2022-05-13 15:44:06
---
# 快速安装
`pip install matplotlib`

# 折线图

## 快速入门


```python
import matplotlib.pyplot as plt
import random

x=range(10) # 定义x轴的数据
y=[random.uniform(15,35) for i in x] # 定义y轴的数据


plt.plot(x, y) # 绘制图像
plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739993.png!webp)
​    


## 设置画布大小

我们使用`plt.figure()`函数来设置画布大小，其参数如下：
- figsize : 设置画布的大小，单位英寸
- dpi : 设置清晰度


```python
import matplotlib.pyplot as plt
import random

x=range(10) # 定义x轴的数据
y=[random.uniform(15,35) for i in x] # 定义y轴的数据

plt.figure(figsize=(20,8),dpi=80) # 设置画布大小与清晰度

plt.plot(x, y) # 绘制图像
plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739995.png!webp)
​    


## 保存图像


```python
plt.savefig('test.png') 
# 在当前目录保存名为test.png的图片，必须在show方法前否则图片就是空白
```

## 自定义x轴、y轴刻度
`xticks`,`yticks`使用自定义刻度的函数，它有两个参数：

- ticks：要显示x轴的刻度
- labels：给对应的x刻度设置一个标签，并且覆盖之前的刻度，与传入ticks的列表长度要相等。


### x轴每隔2两个数显示


```python
import matplotlib.pyplot as plt
import random

x=range(10) # 定义x轴的数据
y=[random.uniform(15,35) for i in x] # 定义y轴的数据

plt.figure(figsize=(20,8),dpi=80) # 设置画布大小与清晰度

plt.xticks(x[::2]) # 定义显示的x轴步长为2

plt.plot(x, y) # 绘制图像
plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739996.png!webp)
​    


### x轴显示中文
matplotlib默认字体是不支持中文的需要更改，有多种方法，现在只提供一种


```python
import matplotlib.pyplot as plt
import random

x=range(10) # 定义x轴的数据
y=[random.uniform(15,35) for i in x] # 定义y轴的数据

plt.figure(figsize=(20,8),dpi=80) # 设置画布大小与清晰度

plt.xticks(x[::2],["1月","2月","3月","4月","5月"]) # 第二个参数可以指定显示字符串，不过传入xticks的这两个参数长度要相等

plt.plot(x, y) # 绘制图像
plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739997.png!webp)
​    


修改matplotlib默认字体，使它支持显示中文


```python
import matplotlib.pyplot as plt
import random

plt.rcParams['font.sans-serif']=['SimHei']  # 用来正常显示中文标签

x=range(10) # 定义x轴的数据
y=[random.uniform(15,35) for i in x] # 定义y轴的数据

plt.figure(figsize=(20,8),dpi=80) # 设置画布大小与清晰度

plt.xticks(x[::2],["1月","2月","3月","4月","5月"]) # 第二个参数可以指定显示字符串，不过传入xticks的这两个参数长度要相等

plt.plot(x, y) # 绘制图像
plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739998.png!webp)
​    


## 轴标签和标题
`xlabel`、`ylabel`使用修改x,y轴标签

`title`可修改标题


```python
import matplotlib.pyplot as plt
import random

plt.rcParams['font.sans-serif']=['SimHei']  # 用来正常显示中文标签

x=range(10) # 定义x轴的数据
y=[random.uniform(15,35) for i in x] # 定义y轴的数据

plt.figure(figsize=(20,8),dpi=80) # 设置画布大小与清晰度

plt.xticks(x[::2],["1月","2月","3月","4月","5月"]) # 第二个参数可以指定显示字符串，不过传入xticks的这两个参数长度要相等

plt.xlabel("时间变化") # 修改标签
plt.ylabel("温度变化")

plt.title("我是标题") # 修改标题

plt.plot(x, y) # 绘制图像
plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739999.png!webp)
​    


## 添加网格线
 `grid`方法来设置图表中的网格线


```python
import matplotlib.pyplot as plt
import random

plt.rcParams['font.sans-serif']=['SimHei']  # 用来正常显示中文标签

x=range(60) # 定义x轴的数据
y=[random.uniform(15,18) for i in x] # 定义y轴的数据

plt.figure(figsize=(20,8),dpi=80) # 设置画布大小与清晰度

plt.plot(x, y) # 绘制图像

plt.xticks(x[::5],["{}分钟".format(i) for i in x][::5]) # 第二个参数可以指定显示字符串，不过传入xticks的这两个参数长度要相等
plt.yticks(range(0,40,5)) # 自定义y轴刻度
plt.xlabel("时间变化") # 修改标签
plt.ylabel("温度变化")

plt.title("我是标题") # 修改标题

# 增加网格显示，0.5表示透明度为50%
plt.grid(linestyle="--",alpha=0.5)


plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739000.png!webp)
​    


## 同一图中同时绘制多条折线
只是数据多准备一份罢了，基本什么区别
例外再添加上图例`legend`，可读性更好


```python
import matplotlib.pyplot as plt
import random

plt.rcParams['font.sans-serif']=['SimHei']  # 用来正常显示中文标签

x=range(60) # 定义x轴的数据
y_1=[random.uniform(15,18) for i in x] # 定义y轴的数据
y_2=[random.uniform(1,3) for i in x] # 定义y轴的数据

plt.figure(figsize=(20,8),dpi=80) # 设置画布大小与清晰度

plt.plot(x, y_1,label="上海") # 绘制图像
plt.plot(x, y_2,label="北京") # 绘制图像

plt.xticks(x[::5],["{}分钟".format(i) for i in x][::5]) # 第二个参数可以指定显示字符串，不过传入xticks的这两个参数长度要相等
plt.yticks(range(0,40,5)) # 自定义y轴刻度

plt.xlabel("时间变化") # 修改标签
plt.ylabel("温度变化")

plt.title("我是标题") # 修改标题

# 增加网格显示，0.5表示透明度为50%
plt.grid(linestyle="--",alpha=0.5)



plt.legend() #绘制图例

plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739001.png!webp)
​    


## 同时绘制两个图
`subplots`可以把整个画布分成多块


```python
plt.rcParams['font.sans-serif']=['SimHei']  # 用来正常显示中文标签
plt.rcParams['axes.unicode_minus'] = False  # 用来正常显示负号

# x,y轴的数据
x=range(60)
y_shanghai=[random.uniform(15,18) for i in x]
# 另一个城市
y_beijing=[random.uniform(1,3) for i in x]



fig, ax = plt.subplots(1,2,figsize=(20,8),dpi=80) # 把画布分成一行两列

ax[0].plot(x,y_shanghai,'b--',label='上海')
ax[1].plot(x,y_beijing,'r',label='北京')

# 图例，必须在plot后面
ax[0].legend()
ax[1].legend()
# 修改x y刻度
x_label=["11点{}分".format(i) for i in range(60)]
ax[0].set_xticks(x[::5],x_label[::5])
ax[1].set_xticks(x[::5],x_label[::5])
ax[0].set_yticks(range(0,40,5))
ax[1].set_yticks(range(0,40,5))
# 增加网格显示
ax[0].grid(linestyle="--",alpha=0.5)
ax[1].grid(linestyle="--",alpha=0.5)

# 添加描述信息
ax[0].set_xlabel('时间变化')
ax[0].set_ylabel('温度变化')
ax[0].set_title('上海城市11点到12点每分钟的温度变化状况')
ax[1].set_xlabel('时间变化')
ax[1].set_ylabel('温度变化')
ax[1].set_title('北京城市11点到12点每分钟的温度变化状况')
plt.show()
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739002.png!webp)
​    


## 绘制数学函数图像


```python
import matplotlib.pyplot as plt
import numpy as np; 

x=np.linspace(-10,10,1000000)
y=x**2

plt.figure(figsize=(8,8),dpi=80)
plt.grid(linestyle='--',alpha=0.5)
plt.plot(x,x**2) 
plt.show()
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739004.png!webp)
​    


## 设置字体大小
在前面的例子中感觉显示的字体太小了，看的不舒服，现在来设置一下。
通过`plt.rcParams['font.size']=18`来指定字体的大小


```python
import matplotlib.pyplot as plt
import random

plt.rcParams['font.sans-serif']=['SimHei']  # 用来正常显示中文标签
plt.rcParams['font.size']=18  # 设置字体大小
x=range(60) # 定义x轴的数据
y=[random.uniform(15,18) for i in x] # 定义y轴的数据

plt.figure(figsize=(20,8),dpi=80) # 设置画布大小与清晰度

plt.plot(x, y) # 绘制图像

plt.xticks(x[::5],["{}分钟".format(i) for i in x][::5]) # 第二个参数可以指定显示字符串，不过传入xticks的这两个参数长度要相等
plt.yticks(range(0,40,5)) # 自定义y轴刻度
plt.xlabel("时间变化") # 修改标签
plt.ylabel("温度变化")

plt.title("我是标题") # 修改标题

# 增加网格显示，0.5表示透明度为50%
plt.grid(linestyle="--",alpha=0.5)


plt.show() # 展示图像
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739005.png!webp)
​    


# 散点图绘制
主要使用`scatter`方法来绘制散点图，参数如下：
参数说明：

- x，y：长度相同的数组，也就是我们即将绘制散点图的数据点，输入数据。
- s：点的大小，默认 20，也可以是个数组，数组每个参数为对应点的大小。
- c：点的颜色，默认蓝色 'b'，也可以是个 RGB 或 RGBA 二维行数组。
- marker：点的样式，默认小圆圈 'o'。
- cmap：Colormap，默认 None，标量或者是一个 colormap 的名字，只有 c 是一个浮点数数组的时才使用。如果没有申明就是 image.cmap。
- norm：Normalize，默认 None，数据亮度在 0-1 之间，只有 c 是一个浮点数的数组的时才使用。
- vmin，vmax：：亮度设置，在 norm 参数存在时会忽略。
- alpha：：透明度设置，0-1 之间，默认 None，即不透明。
- linewidths：：标记点的长度。
- edgecolors：：颜色或颜色序列，默认为 'face'，可选值有 'face', 'none', None。
- plotnonfinite：：布尔值，设置是否使用非限定的 c ( inf, -inf 或 nan) 绘制点。
- **kwargs：：其他参数。


```python
import matplotlib.pyplot as plt
import numpy as np
plt.rcParams['font.size'] = 18
x = np.array([1, 2, 3, 4, 5, 6, 7, 8])
y = np.array([1, 4, 9, 16, 7, 11, 23, 18])
sizes = np.array([20,50,100,200,500,1000,60,90])
plt.figure(figsize=(8,8),dpi=80)
plt.scatter(x,y,s=sizes)
plt.show()
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739006.png!webp)
​    


# 柱状图
主要使用 `bar() `方法来绘制柱形图。
bar() 方法语法格式如下：
- x：浮点型数组，柱形图的 x 轴数据。
- height：浮点型数组，柱形图的高度。
- width：浮点型数组，柱形图的宽度。
- bottom：浮点型数组，底座的 y 坐标，默认 0。
- align：柱形图与 x 坐标的对齐方式，'center' 以 x 位置为中心，这是默认值。 'edge'：将柱形图的左边缘与 x 位置对齐。要对齐右边缘的条形，可以传递负数的宽度值及 align='edge'。
- **kwargs：：其他参数。

## 简单使用


```python
plt.rcParams['font.sans-serif']=['SimHei']  # 用来正常显示中文标签
movie_names = ['雷神3：诸神黄昏','正义联盟','东方快车谋杀案','寻梦环游记','全球风暴', '降魔传','追捕','七十七天','密战','狂兽','其它']
tickets = [73853,57767,22354,15969,14839,8725,8716,8318,7916,6764,52222]
x = range(len(movie_names))

plt.figure(figsize=(20,8),dpi=80)
plt.bar(x,tickets,color=['b','g','r','c','m','y','k'])
plt.xticks(x,movie_names)
plt.title('电影票房收入对比')
plt.grid(linestyle='--',alpha=0.5)
plt.show()

```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739007.png!webp)
​    


## 多组柱状图


```python


plt.rcParams['font.sans-serif']=['SimHei']  # 用来正常显示中文标签
plt.rcParams['axes.unicode_minus'] = False  # 用来正常显示负号

movie_name = ['雷神3:诸神黄昏','正义联盟','寻梦环游记']
first_day = [10587.6,10062.5,1275.7]
first_weekend=[36224.9,34479.6,11830]

plt.figure(figsize=(20,8),dpi=80)

x=range(3)
plt.bar(x,first_day,width=0.2,label='首日票房') # 绘制第一组柱状图
plt.bar([i+0.2 for i in x],first_weekend,width=0.2,label='首周票房') # 绘制第二组柱状图

plt.legend() # 绘制图例

plt.xticks([i+0.1 for i in x],movie_name) # 修改x轴刻度

plt.show()
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739008.png!webp)
​    


## 垂直水平方向的柱状图
垂直方向的柱状图可以使用`barh()` 方法来设置：


```python
import matplotlib.pyplot as plt

x = ["Runoob-1", "Runoob-2", "Runoob-3", "C-RUNOOB"]
y = [12, 22, 6, 18]
plt.figure(figsize=(20,8),dpi=80)
plt.barh(x,y)
plt.show()
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739009.png!webp)
​    


# 直方图
使用`hist()`方法来绘制直方图


```python
# 电影时长分布状况
time = [131,  98, 125, 131, 124, 139, 131, 117, 128, 108, 135, 138, 131, 102, 107, 114, 119, 128, 121, 142, 127, 130, 124, 101, 110, 116, 117, 110, 128, 128, 115,  99, 136, 126, 134,  95, 138, 117, 111,78, 132, 124, 113, 150, 110, 117,  86,  95, 144, 105, 126, 130,126, 130, 126, 116, 123, 106, 112, 138, 123,  86, 101,  99, 136,123, 117, 119, 105, 137, 123, 128, 125, 104, 109, 134, 125, 127,105, 120, 107, 129, 116, 108, 132, 103, 136, 118, 102, 120, 114,105, 115, 132, 145, 119, 121, 112, 139, 125, 138, 109, 132, 134,156, 106, 117, 127, 144, 139, 139, 119, 140,  83, 110, 102,123,107, 143, 115, 136, 118, 139, 123, 112, 118, 125, 109, 119, 133,112, 114, 122, 109, 106, 123, 116, 131, 127, 115, 118, 112, 135,115, 146, 137, 116, 103, 144,  83, 123, 111, 110, 111, 100, 154,136, 100, 118, 119, 133, 134, 106, 129, 126, 110, 111, 109, 141,120, 117, 106, 149, 122, 122, 110, 118, 127, 121, 114, 125, 126,114, 140, 103, 130, 141, 117, 106, 114, 121, 114, 133, 137,  92,121, 112, 146,  97, 137, 105,  98, 117, 112,  81,  97, 139, 113,134, 106, 144, 110, 137, 137, 111, 104, 117, 100, 111, 101, 110,105, 129, 137, 112, 120, 113, 133, 112,  83,  94, 146, 133, 101,131, 116, 111,  84, 137, 115, 122, 106, 144, 109, 123, 116, 111,111, 133, 150]
plt.figure(figsize=(20,8),dpi=80)

distince = 2 # 每组的间距
plt.hist(time,(max(time)-min(time))//distince)
plt.xticks(range(min(time),max(time)+2,distince))

plt.grid(linestyle='--',alpha=0.5)
plt.xlabel('电影时长大小')
plt.ylabel('电影的数据量')
plt.title("电影时长分布")
plt.show()
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739010.png!webp)
​    


# 饼图
使用 pyplot 中的 `pie()` 方法来绘制饼图。
参数说明：

- x：浮点型数组，表示每个扇形的面积。
- explode：数组，表示各个扇形之间的间隔，默认值为0。
- labels：列表，各个扇形的标签，默认值为 None。
- colors：数组，表示各个扇形的颜色，默认值为 None。
- autopct：设置饼图内各个扇形百分比显示格式，%d%% 整数百分比，%0.1f 一位小数， %0.1f%% 一位小数百分比， %0.2f%% 两位小数百分比。
- labeldistance：标签标记的绘制位置，相对于半径的比例，默认值为 1.1，如 <1则绘制在饼图内侧。
- pctdistance：：类似于 labeldistance，指定 autopct 的位置刻度，默认值为 0.6。
- shadow：：布尔值 True 或 False，设置饼图的阴影，默认为 False，不设置阴影。
- radius：：设置饼图的半径，默认为 1。
- startangle：：起始绘制饼图的角度，默认为从 x 轴正方向逆时针画起，如设定 =90 则从 y 轴正方向画起。
- counterclock：布尔值，设置指针方向，默认为 True，即逆时针，False 为顺时针。
- wedgeprops ：字典类型，默认值 None。参数字典传递给 wedge 对象用来画一个饼图。例如：wedgeprops={'linewidth':5} 设置 wedge 线宽为5。
- textprops ：字典类型，默认值为：None。传递给 text 对象的字典参数，用于设置标签（labels）和比例文字的格式。
- center ：浮点类型的列表，默认值：(0,0)。用于设置图标中心位置。
frame ：布尔类型，默认值：False。如果是 True，绘制带有表的轴框架。
rotatelabels ：布尔类型，默认为 False。如果为 True，旋转每个 label 到指定的角度。


```python
movie_name = ['雷神3:诸神黄昏','正义联盟','东方快车谋杀案','寻梦环游记','全球风暴','降魔传','追捕','七十七天','密战','狂兽','其它']
place_count = [60605,54546,45819,28243,13270,9945,7679,6799,6101,4621,20105]
plt.figure(figsize=(20,8),dpi=80)
plt.pie(place_count,labels=movie_name,autopct='%1.2f%%')
plt.legend()
plt.axis('equal') # 确保饼图能化成一个圆
plt.show()
```


​    
![png](https://halo-1257208482.image.myqcloud.com/202205141739011.png!webp)
​    


