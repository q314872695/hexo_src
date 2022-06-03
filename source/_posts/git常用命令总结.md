---
title: git常用命令总结
date: 2022-06-02 17:37:34
tags: git
---



# git全局设置

**设置用户信息**

```bash
git config --global user.name "用户名"
git config --global user.email "邮箱"
```

> 上面设置的user.name和user.email并不是我们在注册码云或github账号时使用的用户名和邮箱，此处可以任意设置。

**查看配置信息**

```bash
git config --list
```



# 获取git仓库

**本地初始化一个git仓库**

```bash
git init # 会在当前目录创建一个本地仓库
```

**从远程仓库克隆一个仓库**

```bash
git clone 远程仓库地址
```



# git基本操作

## git add

-  `git add hello.txt` :  将hello.txt 添加到暂存区
- `git add .` : 将当前目录下的所有文件添加到暂存区

## git commit

- `git commit -m [message]`：把暂存去的文件提交到本地仓库
- `git commit -am [message]`：把文件直接提交到本地仓库，不需要git add

## git status

该命令用于查看文件状态

## git reset

git reset 命令用于将暂存区的文件取消暂存或者退回某一次提交的版本。

- `git reset index.html` ：将index.html取消暂存
- `git reset --hard 25ddcd44bf0a2daf6526647bebb5be537dcb2d77 ` ：退回到指定版本，后面的一推字符串是某一版本的唯一标识，通过`git log`查看
-  `git reset --hard origin/master`：将本地的状态回退到和远程的一样 



## git log

查看提交的日志

# 远程操作

## git remote

**git remote** 命令用于在远程仓库的操作。

- 查看远程仓库

```bash
git remote -v # 查看远程仓库
gitee   https://gitee.com/lxy777/test.git (fetch)
gitee   https://gitee.com/lxy777/test.git (push)

git remote show gitee # 查看某个远程仓库的信息
* remote gitee
  Fetch URL: https://gitee.com/lxy777/test.git
  Push  URL: https://gitee.com/lxy777/test.git
  HEAD branch: master
  Remote branches:
    dev    tracked
    master new (next fetch will store in remotes/gitee)
  Local refs configured for 'git push':
    dev    pushes to dev    (up to date)
    master pushes to master (local out of date)
```

- 添加远程仓库


```bash
git remote add [别名] [url] 添加远程仓库
git remote origin https://gitee.com/lxy777/test.git # 一般习惯写origin，也可以起别的名字
```

- 删除远程仓库

```bash
git remote rm name  # 删除远程仓库
```

## git clone

克隆远程仓库到本地

命令格式如下：

```bash
git clone [url] # 将远程仓库克隆到本地
```

## git push

上传文件到远程仓库

命令格式如下：

```bash
git push <远程主机名> <本地分支名>:<远程分支名>
```

如果本地分支名与远程分支名相同，则可以省略冒号：

```bash
git push <远程主机名> <本地分支名>
```

## git pull

**git pull** 命令用于从远程获取最新代码并合并本地的版本。

命令格式如下：

```bash
git pull <远程主机名> <远程分支名>:<本地分支名>

git pull origin master:brantest
# 将远程主机 origin 的 master 分支拉取过来，与本地的 brantest 分支合并。
```

如果远程分支与当前分支合并，则冒号后面的可以省略

```bash
git pull origin master
```

**注意:**如果当前本地仓库不是从远程仓库克隆，而是本地创建的仓库，并且仓库中存在文件，此时再从远程仓库拉取文件的时候会报错(fatal: refusing to merge unrelated histories )
解决此问题可以在git pull命令后加入参数`--allow-unrelated-histories`

# 分支管理

## 查看分支

查看本地分支

```bash
git branch
```

查看远程分支

```bash
git branch -r
```

查看本地分支和远程仓库分支

```bash
git branch -a
```



## 创建分支

```bash
git branch [分支名称]
```

## 切换分支

```bash
git checkout [分支名称]
```

## 合并分支

将指定分支合并到当前分支

```bash
git merge [分支名称]
```

## 删除分支

```bash
git branch -d [分支名称]
```

# 标签管理

## 查看标签

```bash
git tag
```

## 创建标签

```bash
git tag [标签名]
```

## 推送标签至远程仓库

```bash
git push [远程仓库名] [标签名]
```

## 检出标签

```bash
git checkout -b [新分支名] [标签名]
```



