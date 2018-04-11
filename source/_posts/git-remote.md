---
title: Git笔记
date: 2018-02-02 13:58:28
tags:
---

![pic](http://www.wailian.work/images/2018/02/06/e58c7755f88b1e0ad82a30100cd3f781e3fe8f5970ada-GGAkXt_fw658.jpg)

# 初始化项目

## 在GitHub或者码云上新建仓库

登录GitHub或者码云，新建仓库，不默认产生README文档

## 本地新建仓库

```
git init
```

## 本地仓库与远程相连

```
git remote add origin git@git.oschina.net:wangweiye/SpringBoot-Learning.git
```

## 提交本地到远程

```
git add .

git commit -m ""

git pull origin master

git push -u origin master
```

# 标签

## 列显已有的标签

```
git tag
```

## 新建轻量级标签

```
git tag v1.0.0
```

## 分享标签

```
git push origin [tagname]
```

## 删除远程tag

```
git push origin --delete tag [tagname]
```

# .gitignore无效解决方法

在工程中很容易出现.gitignore并没有忽略掉我们已经添加的文件，那是因为.gitignore对已经追踪(track)的文件是无效的，需要清除缓存，清除缓存后文件将以未追踪的形式出现，这时重新添加(add)并提交(commit)就可以了。

```
git rm -r --cached .
git add .
git commit -m "comment"
```
