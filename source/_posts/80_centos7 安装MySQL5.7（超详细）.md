---
title: centos7 安装MySQL5.7（超详细）
tags: mysql
abbrlink: 5a6a3a95
date: 2020-05-29 17:24:24
---

> 之前一直用的宝塔安装的软件，现在用命令行安装一下，遇到很多坑来记录一下,本文参考mysql官方文档https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html

# 添加MySQL Yum存储库
1. 打开网址`http://repo.mysql.com/`

![image.png](https://halo-1257208482.image.myqcloud.com/image_1590736107586.png!webp)

2. 适合自己电脑版本的rpm文件
	1. 对于基于EL6(centos 6)的系统，rpm包形式为：`mysql57-community-release-el6-{version-number}.noarch.rpm`
	2. 对于基于EL6(centos 7)的系统，rpm包形式为：`mysql57-community-release-el7-{version-number}.noarch.rpm`
	3. 对于基于EL6(centos 8)的系统，rpm包形式为：`mysql57-community-release-el7-{version-number}.noarch.rpm`

3. 下载rpm文件`wget http://repo.mysql.com/mysql57-community-release-el7-9.noarch.rpm`，对于没有wget 命令的可以先`yum install wget`
4. `rpm -ivh http://repo.mysql.com/mysql57-community-release-el7-9.noarch.rpm` 


![image.png](https://halo-1257208482.image.myqcloud.com/image_1590735743114.png!webp)

5. 可以使用`yum repolist enabled | grep "mysql.*-community.*"`检查是否已成功添加MySQL Yum存储库

![image.png](https://halo-1257208482.image.myqcloud.com/image_1590735831753.png!webp)

6. 对用mysql8.0的安装也是同样，只要找到对应的rpm包即可。格式：`mysql80-community-release-el6-{version-number}.noarch.rpm`
# 安装mysql
通过以下命令安装MySQL：
- `sudo yum -y install mysql-community-server`

# 启动MySQL服务器
- 启动MySQL服务器：`sudo service mysqld start`
- 使用以下命令检查MySQL服务器的状态：`sudo service mysqld status`,正常情况应该如图下所示

![image.png](https://halo-1257208482.image.myqcloud.com/image_1590741749108.png!webp)

# 细节

**假设服务器的数据目录为空，则在服务器首次启动时，会发生以下情况：**

- 服务器已初始化。

- SSL证书和密钥文件在数据目录中生成。

- `validate_password`已安装并启用。实施的默认密码策略要求密码至少包含一个大写字母，一个小写字母，一位数字和一个特殊字符，并且密码总长度至少为8个字符。

- `'root'@'localhost`创建 一个超级用户帐户。设置超级用户的密码并将其存储在错误日志文件中。要显示它，请使用以下命令：`sudo grep 'temporary password' /var/log/mysqld.log`


![image.png](https://halo-1257208482.image.myqcloud.com/image_1590741906407.png!webp)

**通过使用生成的临时密码登录并尽快为超级用户帐户设置自定义密码，以更改root密码：**
- `mysql -uroot -p`
- `ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';`

**如果是用于学习，可以把密码策略validate_password关闭设置个简单的密码：**
- 打开配置文件`vi /etc/my.cnf`
- 在配置文件末尾添加上`validate_password = off`
- 重启mysql服务器

**默认情况下，mysql 只允许本机访问也就是localhost,可以通过一下方法开启远程访问：**
- 创建一个账户：`grant all on *.* to '你的用户名'@'%' identified by '你的密码' with grant option;` 
	- 语法解释：`grant 权限列表 on 数据库名.表名 to '用户名'@'主机名';`
	- `with grant option`允许给其他人赋予自己拥有的权限
- 如果还是无法访问那就是防火墙的问题`service firewalld stop`关闭就好了

