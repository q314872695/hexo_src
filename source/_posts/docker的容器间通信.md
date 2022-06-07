---
title: docker的容器间通信
date: 2022-06-06 21:53:09
tags: docker
---



# 引言

在使用docker的过程中，每创建一个容器都会为其分配一个ip地址，如果直接使用ip地址通讯的话，虽然能够通讯，但也太麻烦了，最重要的是这个ip地址还会变化，下次重启容器说不准程序就跑不通了，与是乎就出现了以下两种通信的方式，其本质上也是通过访问容器名称这个域名经过docker的dns服务器自动解析成ip地址。

# 使用--link

这种方式适用于，已经有一个容器正在运行，在第二个容器启动时加上参数`--link 容器名称`，就可以访问第一个正在运行的容器了。

**例如：**

当前有一个容器名称为centos_1的容器正在运行

```bash
lxy@DESKTOP-74EDKD3:~$ docker ps
CONTAINER ID   IMAGE            COMMAND       CREATED        STATUS         PORTS     NAMES
7e0c0f7a46f9   centos:centos7   "/bin/bash"   17 hours ago   Up 2 seconds             centos_1
```

现要创建另一个容器去连接第一个容器

```bash
docker run -id --name centos_2  --link centos_1 centos:centos7
```

- `--link centos_1` ：表示当前要创建的容器要连接centos_1这个容器

**测试：**

```bash
lxy@DESKTOP-74EDKD3:~$ docker exec -it centos_2 bash # 进入centos_2容器
[root@2879a10f92f1 /] ping centos_1 # 访问centos_1容器
PING centos_1 (172.17.0.2) 56(84) bytes of data.
64 bytes from centos_1 (172.17.0.2): icmp_seq=1 ttl=64 time=0.133 ms
64 bytes from centos_1 (172.17.0.2): icmp_seq=2 ttl=64 time=0.074 ms
64 bytes from centos_1 (172.17.0.2): icmp_seq=3 ttl=64 time=0.043 ms
64 bytes from centos_1 (172.17.0.2): icmp_seq=4 ttl=64 time=0.039 ms
64 bytes from centos_1 (172.17.0.2): icmp_seq=5 ttl=64 time=0.045 ms
^C
--- centos_1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4172ms
rtt min/avg/max/mdev = 0.039/0.066/0.133/0.036 ms
```



# 使用自定义网络

docker默认的`bridge`网络是不能把容器名称作为ip进行通讯的，只有自定义网络才行。在自定义网络中的容器之间可以相互通讯。

- 创建自定义网络

```bash
docker network create -d bridge my-net
# y
docker network create my-net  # 参数可以省略，默认创建的就是桥接类型的网络
```

- 启动容器时明确指定使用哪个网络

```bash
docker run -id --name centos_2  --network my-net centos:centos7
```

- 启动容器后加入到某个网络

```bash
docker network connect my-net centos_1 # 把容器centos_1加入自定义网络my-net
```

