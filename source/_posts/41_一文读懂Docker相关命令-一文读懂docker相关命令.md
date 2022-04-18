---
title: 一文读懂Docker相关命令
tags: docker
abbrlink: 3b19f091
date: 2020-04-13 22:05:14
---

![](https://halo-1257208482.image.myqcloud.com/202204051744892.jpeg!webp)

<!--more-->

以下命令以centos为例

### 进程相关命令

- 启动docker服务

```bash
systemctl start docker
```

- 停止docker服务

```bash
systemctl start docker
```

- 重启docker服务

```bash
systemctl restart docker
```

- 查看docker服务状态

```bash
systemctl status docker
```

- 设置开机启动docker服务

```bash
systemctl enable docker
```

### 镜像相关命令

- 查看镜像：查看本地所有的镜像

```bash
docker images
docker images –q # 查看所用镜像的id
```

- 搜索镜像：从网络中查找需要的镜像，即使已经配置了镜像加速，它还是会从hub.docker.com上搜索，有时可能会很慢。

```bash
docker search 镜像名称
```

- 拉取镜像：从Docker仓库下载镜像到本地，镜像名称格式为 名称:版本号，如果版本号不指定则是最新的版本。如果不知道镜像版本，可以去docker hub 搜索对应镜像查看。

```bash
docker pull 镜像名称
```

- 删除镜像

```bash
docker rmi 镜像id # 删除指定本地镜像
docker rmi `docker images -q`  # 删除所有本地镜像
```

### 容器相关命令

- 查看容器

```bash
docker ps # 查看正在运行的容器
docker ps –a # 查看所有容器
```

- 创建并启动容器

```bash
docker run 参数 镜像名称
```

参数说明：

1. -i：保持容器运行。通常与-t同时使用。加入-it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。
2. -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用。
3. -d：以守护（后台）模式运行容器。创建一个容器在后台运行，需要使用docker exec 进入容器。退出后，容器不会关闭。
4. -it：创建的容器一般称为交互式容器，-id：创建的容器一般称为守护式容器。
5. --name=名字：为创建的容器命名。
6. -v ：设置数据卷（即目录映射）必须是绝对路径，如果目录不存在会自动创建。例如`-v 宿主机目录(文件):容器内目录(文件)`。若使用`–v /volume` 则表示把该容器设置为数据卷容器。
7. -p ：宿主机和容器间的端口映射。例如`-p 宿主机端口:容器端口`。

- 进入容器

```bash
docker exec 参数 容器名称或者容器id # 退出容器，容器不会关闭，参数通常为-it
```

- 停止容器

```bash
docker stop 容器名称
```

- 启动容器

```bash
docker start 容器名称
```

- 删除容器

```bash
docker rm 容器名称
```

- 查看容器信息

```bash
docker inspect 容器名称
```

