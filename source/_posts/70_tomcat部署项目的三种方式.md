---
title: tomcat部署项目的三种方式
date: 2020-05-21 15:42:32
tags:
- java
- tomcat
---

## 直接将项目放到webapps目录下即可
* /hello：项目的访问路径-->虚拟目录
* 简化部署：将项目打成一个war包，再将war包放置到webapps目录下。
	* war包会自动解压缩

## 配置conf/server.xml文件
- 在`<Host>`标签体中配置`<Context docBase="D:\hello" path="/hehe" />`
	* docBase:项目存放的路径
	* path：虚拟目录

## 在conf\Catalina\localhost创建任意名称的xml文件。
- 在文件中编写`<Context path="/login_demo" docBase="E:\idea\login_demo\out\artifacts\login_demo_war_exploded" />`
	* docBase:项目存放的路径
	* path：虚拟目录

