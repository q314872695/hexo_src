---
title: h5中的分组元素figure、figcaption、hgroup元素介绍
date: 2019-09-15 15:24:00
tags: html
---



分组元素用于对页面中的内容进行分组。

# figure元素和figcaption元素
- **figure**元素用于定义独立的流内容（图像、图表、照片、代码等），一般指一个独立的单元。**figure**元素的内容应该与主内容相关，但如果被删除，也**不会对文档流产生影响**。
- **figcaption**元素用于为figure元素组添加标题，**一个figure元素内最多允许使用一个figcaption元素**，该元素应该放在figure元素的第一个或最后一个子元素的位置。
```html
<p>被称作“第四代体育馆”</p>
<figure>
    <figcaption>北京鸟巢</figcaption>
    <p>拍摄者：张三，拍摄时间：2019年9月15日 17:42:12</p>
    <img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1568550641218&di=3d4dc50f7b0fe9a2ed3a8b25198b3730&imgtype=0&src=http%3A%2F%2Fm.tuniucdn.com%2Ffb2%2Ft1%2FG1%2FM00%2F0C%2F9D%2FCii9EFbQJa2IV2ubAAIUZv3TCucAACUQgCQyfAAAhR-415_w500_h280_c1_t0.jpg"
         alt="">
</figure>
```
# hgroup元素
hgroup元素用于将多个标题（主标题和副标题或者子标题）组成一个标题组，通常它与h1~h6元素组合使用。通常，将hgroup元素放在header元素中。
在使用hgroup元素时要注意以下几点：
- 如果只有一个标题元素不建议使用hgroup元素。
- 当出现一个或者一个以上的标题与元素时，推荐使用hgroup元素作为标题元素。
- 当一个标题包含副标题、section或者article元素时，建议将hgroup元素和标题相关元素存放到header元素容器中。

```html
<header>
    <hgroup>
        <h1>我的个人网站</h1>
        <h2>我的个人作品</h2>
    </hgroup>
    <p>开心快乐每一天</p>
</header>
```
