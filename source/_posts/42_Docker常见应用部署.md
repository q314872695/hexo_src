---
title: Docker常见应用部署
tags: docker
abbrlink: e3883321
date: 2020-04-15 21:37:53
---


记录了docker中安装mysql,tomcat,nginx,redis,anaconda的安装方式，以便以后方便使用。

<!--more-->

# 一、部署MySQL

1. 搜索mysql镜像

```shell
docker search mysql
```

2. 拉取mysql镜像

```shell
docker pull mysql:5.6
```

3. 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql
cd ~/mysql
```

```shell
docker run -id \
-p 3306:3306 \
--name=c_mysql \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.6
```

- 参数说明：
  - $PWD：表示当前目录所在路径，现在表示`/root/mysql`。
  - **-p 3306:3306**：将容器的 3306 端口映射到宿主机的 3306 端口。
  - **-v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。配置目录
  - **-v $PWD/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。日志目录
  - **-v $PWD/data:/var/lib/mysql** ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。数据目录
  - **-e MYSQL_ROOT_PASSWORD=123456：**初始化 root 用户的密码。
  - 创建容器后会自动启动mysql服务，直接使用即可。

4. 进入容器，操作mysql。

```shell
docker exec –it c_mysql /bin/bash
```

5. 使用外部机器连接容器中的mysql。

![](https://halo-1257208482.image.myqcloud.com/202204051744253.jpeg!webp)


# 二、部署Tomcat

1. 搜索tomcat镜像

```shell
docker search tomcat
```

2. 拉取tomcat镜像

```shell
docker pull tomcat
```

3. 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建tomcat目录用于存储tomcat数据信息
mkdir ~/tomcat
cd ~/tomcat
```

```shell
docker run -id --name=c_tomcat \
-p 8080:8080 \
-v $PWD:/usr/local/tomcat/webapps \
tomcat 
```

- 参数说明：
  - **-p 8080:8080：**将容器的8080端口映射到主机的8080端口
  - **-v $PWD:/usr/local/tomcat/webapps：**将主机中当前目录挂载到容器的webapps
  - 创建容器后会自动自动tomcat服务，直接使用即可。

4. 使用外部机器访问tomcat


# 三、部署Nginx

1. 搜索nginx镜像

```shell
docker search nginx
```

2. 拉取nginx镜像

```shell
docker pull nginx
```

3. 创建容器，设置端口映射、目录映射


```shell
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx
cd ~/nginx
mkdir conf
cd conf
# 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容(nginx默认的主配置文件)
vim nginx.conf
```
```shell

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}


```




```shell
docker run -id --name=c_nginx \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html \
nginx
```

- 参数说明：
  - **-p 80:80**：将容器的 80端口映射到宿主机的 80 端口。
  - **-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf**：将主机当前目录下的 /conf/nginx.conf 挂载到容器的 :/etc/nginx/nginx.conf。配置目录
  - **-v $PWD/logs:/var/log/nginx**：将主机当前目录下的 logs 目录挂载到容器的/var/log/nginx。日志目录
  - 创建容器后会自动启动nginx服务，直接使用即可。

4. 使用外部机器访问nginx

# 四、部署Redis

1. 搜索redis镜像

```shell
docker search redis
```

2. 拉取redis镜像

```shell
docker pull redis:5.0
```

3. 创建容器，设置端口映射

```shell
docker run -id --name=c_redis -p 6379:6379 redis:5.0
```

4. 使用外部机器连接redis

# 五、部署Anaconda

1. 拉取anaconda镜像

```bash
# 安装python3版本的
docker pull continuumio/anaconda3
```

2. 创建容器，设置端口映射

```bash
docker run -it --name=anaconda -p 8888:8888 continuumio/anaconda3
```

参数说明：

- -p 8888:8888：将宿主机的8888端口和容器8888端口进行映射
- -it：启动后即进入容器终端

3. 在容器中启动jupyter notebook服务

```bash
jupyter notebook --ip=0.0.0.0 --allow-root
```

参数说明：

- --ip=0.0.0.0：允许任何用户访问
- --allow-root：允许以root用户启动服务，因为当前例子是以root用户登录的，没有这句会报错

4. 复制命令行中的地址，用浏览器访问

![](https://pic.downk.cc/item/5e96d6d1c2a9a83be5a74f1a.jpg)