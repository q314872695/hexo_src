---
title: node版本冲突怎么办？快使用nvm-windows
date: 2022-01-19 17:21:38
tags: node
---

# 前言

一般来说安装软件都喜欢安装最新版本的，我当然也不例外，电脑上已经有了最新版本的node了，最近看了个vue项目，由于使用版本可能比较老，在使用`npm install` 时死活不行，后来百度一下才知道时node版本太高了，需要换个低版本的，为了一个旧项目也不值得来回安装卸载node了，就在想有没有一个关于node版本管理的软件呢，后来在一番搜索下找到了一款神器`nvm`

# 介绍

[nvm](https://github.com/creationix/nvm)是 Mac 下的 node 管理工具，如果需要管理 Windows 下的 node，官方推荐使用 [nvm-windows](https://github.com/coreybutler/nvm-windows)。不过，nvm-windows 并不是 nvm 的简单移植，他们也没有任何关系，但它们的命令基本是一样的。题主电脑是windows系统 ，就只好简单演示 **nvm-windows**的使用了。

# 安装

截止到现在nvm-windows的最新版本是1.1.9，点击[下载](https://github.com/coreybutler/nvm-windows/releases)，即可。**需要注意的是安装nvm-windows之前需要卸载之前的node**，把之前相关文件夹都删干净了就可以无脑安装了。如果在`cmd`中输入`nvm`出现如下所示则说明安装成功了。

![](https://pic.imgdb.cn/item/61e90ef92ab3f51d913ac529.jpg)

# 使用

**安装node**

 `nvm install <version>`：安装一个指定版本的node，直接指定版本号即可。

 `nvm install latest `：安装最新本版。

**使用node**

`nvm use <version> ` ：使用指定版本node

`nvm use latest ` ：使用最新版本node

**查看node**

`nvm list`：查看所有的node版本

`nvm current`：查看当前正在使用的node版本

**卸载node**

`nvm uninstall <version>`：卸载指定版本的node

**设置镜像**

`nvm node_mirror <node_mirror_url>`: 设置node镜像

`nvm npm_mirror <npm_mirror_url>`: 设置npm镜像

（默认镜像就下载的挺快的，我就没设置）
