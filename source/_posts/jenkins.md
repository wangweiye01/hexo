---
title: Jenkins部署Spring Boot
date: 2017-12-15 23:00:10
tags:
---
# 持续集成是什么？

互联网软件的开发和发布，已经形成了一套标准流程，最重要的组成部分就是持续集成（Continuous integration，简称CI）

# 持续集成的目的

让产品可以快速迭代，同时还能保持高质量

# Jenkins是什么？

Jenkins是一个用Java编写的开源的持续集成工具，因此安装Jenkins必须有Java运行环境

# Jenkins安装

[官网](https://jenkins.io/download/)下载Jenkins安装包，按照官网提示进行安装

## 安装常用插件
![常用插件](http://s1.wailian.download/2017/12/18/1eebae.png)
## 创建密码
![创建密码](http://s1.wailian.download/2017/12/18/2625d8.png)

注意：需要防火墙打开8080端口，Jenkins默认使用8080端口

# Jenkins的配置

## 全局工具配置

- 配置jdk
![](http://s1.wailian.download/2017/12/18/2ad234.png)
- 配置maven
![](http://s1.wailian.download/2017/12/18/3711fb.png)

## 插件安装

Jenkins有很多插件已经被安装，其中Git plugin和Maven Integration plugin，publish over SSH是部署Spring Boot项目必备的插件

## 配置Credentials

配置成功后可以用ssh协议拉取git上的代码

![](http://s1.wailian.download/2017/12/15/1.png)
