---
title: docker数据卷管理
date: 2022-06-08 16:16:19
tags: docker
---

# 什么是数据卷

- 数据卷是宿主机中的一个目录或文件
- 当容器目录和数据卷目录绑定后，对方的修改会立即同步
- 一个数据卷可以被多个容器同时挂载
- 一个容器也可以被挂载多个数据卷

# 数据卷的作用

- 容器数据持久化
- 外部机器和容器间接通信
- 容器之间数据交换

# 如何使用数据卷

## 使用绝对路径数据卷

```bash
docker run -id -v /宿主机的路径:/容器内的路径 镜像名
```

**注意：** 宿主机路径必须是绝对路径,宿主机目录会覆盖容器内目录内容

## 使用别名方式数据卷

```bash
docker run -id -v 别名:/容器内的路径 镜像名
# 例如
docker run -id --name mysql \
	-v mysql_data:/var/lib/mysql \
	-p 3306:3306 \
	mysql:8
```

**注意：**

- `mysql_data`代表一个数据卷别名，别名可以是任意的。
- `mysql_data`这个数据别名可以省略，省略的时候docker会自动生成一串随机序列来充当这个数据卷的别名。
- docker首次用到这个数据卷时会自动创建，数据卷已经存在时会使用原来的。
- 第一次使用别名时会将容器中原始数据保留下来，使用绝对路径方式不会保留容器中原始数据
- 

```bash
# 查看所有数据卷
lxy@DESKTOP-74EDKD3:~$ docker volume ls
DRIVER    VOLUME NAME
local     456a70b64155be838c487db08ac50852060618d7cc6a16fb5275f80b6961d064
local     b91ad14c73d03b5cffd5053592f7a6c80d6294ef4b0146c55ee943008d1d0612
local     portainer_data

# 查看数据卷详细信息
lxy@DESKTOP-74EDKD3:~$ docker volume inspect portainer_data
[
    {
        "CreatedAt": "2022-06-08T07:31:45Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/portainer_data/_data",
        "Name": "portainer_data",
        "Options": null,
        "Scope": "local"
    }
]
#/var/lib/docker/volumes/portainer_data/_data  这个路径是数据卷在宿主机的路径

# 删除指定数据卷
docker volume rm 数据卷别名

# 删除未被使用的数据卷
docker volume prune

# 手动创建一个数据卷
docker volume create 数据卷别名 # 在docker run中-v mysql_data:/var/lib/mysql会自动创建这个数据卷，手动创建还多操作了一步，麻烦
```

