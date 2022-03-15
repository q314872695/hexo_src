---
title: h5中的结构元素header、nav、article、aside、section、footer介绍
tags: html
abbrlink: 342a8a7
date: 2019-09-13 15:24:00
---



结构元素不具有任何样式，只是使页面元素的的语义更加明确。**

# header元素
header元素是一种具有引导和导航作用的的结构元素，**该元素可以包含所有通常放在页面头部的内容**。header元素通常用来放置整个页面或页面内的一个内容区块的标题，也可以包含网站Logo图片、搜索表单或者其他相关内容。
```html
<header>
	<h1>网页主题</h1>
</header>
```
一个网页中可以使用多个header元素，也可以为每一个内容块添加header元素。
# nav元素
nav元素用于定义导航链接，是html5新增的元素。该元素可以**将具有导航性质的链接归纳在一个区域中**，使页面元素的语义更加明确。

nav元素通常可以用于以下场合中：
- 传统导航条
- 侧边栏导航
- 页内导航
- 翻页操作
```html
<nav>
	<ul>
		<li><a href="#">首页</a></li>
		<li><a href="#">公司概况</a></li>
		<li><a href="#">产品展示</a></li>
		<li><a href="#">联系我们</a></li>
	</ul>
</nav>
```
# article元素
article元素代表文档、页面或者应用程序中与上下文不相关的独立部分，该元素经常被用于定义一篇日志、一条新闻或用户评论等。article元素通常使用多个section元素进行划分，一个页面中article元素可以出现多次。
```html
<article>
    <header>
        <h1>第一章</h1>

    </header>
    <section>
        <header>
            <h2>第1节</h2>
        </header>
    </section>
    <section>
        <header>
            <h2>第2节</h2>
        </header>
    </section>
</article>
```
# aside元素
aside元素用来定义当前页面或者文章的复数信息部分，它可以包含与当前页面或主要内容相关的引用、侧边栏、广告、导航条等其他类似的有别于主要内容的部分。

aside元素主要的用法分为两种：
- 被包含在article元素内作为主要内容的附属信息。
- 在article元素之外使用，作为页面或者站点全局的附属信息部分。最常用的形式是侧边栏，其中的内容可以使友情链接、广告单元等。

```html
<article>
    <header>
        <h1>标题</h1>
    </header>
    <section>文章主要内容</section>
    <aside>其他相关内容</aside>
</article>
<aside>右侧菜单</aside>
```
# section元素
section元素用于对网站或应用程序中页面上的内容进行分块，一个section元素通常由内容和标题组成。在使用section元素时，需要注意一下三点：
- 不要将section元素用作设置样式的页面容器，那是div的特性。
- 如果article元素、aside元素或nav元素更符合使用条件，那么不要使用section元素。
- 没有标题的内容区块不要使用section元素定义。

```html
<article>
    <header>
        <h2>小张的个人介绍</h2>
    </header>
    <p>小张是一个好学生，是一个帅哥。。。</p>
    <section>
        <h2>评论</h2>
        <article>
            <h3>评论者：A</h3>
            <p>小张真的很帅</p>
        </article>
        <article>
            <h3>评论者：B</h3>
            <p>小张是一个好学生</p>
        </article>
    </section>
</article>
```
在html5中，article元素可以看作是一种特殊的section元素，它比section元素更具有独立性，即section元素强调分段或分块，而article元素强调独立性。如果一块内容相对来说比较独立、完整时，应该使用article元素；但是如果想要将一块内容分成多段时，应该使用section元素。

# footer元素
footer元素用于定义一个页面或者区域的底部，它可以包含所有通常放在页面底部的内容。与header元素相同，一个页面中可以包含多个footer元素。同时，也可以在article元素或者section元素中添加footer元素。
```html
<article>
    文章内容
    <footer>
        文章分页列表
    </footer>
</article>
<footer>
    页面底部
</footer>
```
