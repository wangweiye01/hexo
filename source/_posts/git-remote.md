---
title: Git笔记
date: 2018-02-02 13:58:28
tags:
---
![](http://www.wailian.work/images/2018/02/06/d95260be85e5b41446a25914860307cbfa056a2c370c3-ZkMgle_fw658.th.jpg)

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
