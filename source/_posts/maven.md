---
title: maven构建Spring Boot多模块项目以及打包方法
date: 2018-03-26 15:06:22
tags:
---
![](http://www.wailian.work/images/2018/03/28/f5462f69549047be1b086629e548eefc8d74eca329bea-IiKAbw_fw658.jpg)

> 比起传统复杂的单体工程，使用Maven的多模块配置，可以帮助项目划分模块，鼓励重用，防止POM变得过于庞大，方便某个模块的构建，而不用每次都构建整个项目，并且使得针对某个模块的特殊控制更为方便

# 构建项目

## 创建空的maven主项目

![create](http://www.wailian.work/images/2018/03/26/1.png)

![mvn](http://www.wailian.work/images/2018/03/26/2.png)

## 删除src目录

![src](http://www.wailian.work/images/2018/03/26/3.png)

## 修改pom文件

添加`<packaging>pom</packaging>`

![](http://www.wailian.work/images/2018/03/26/4.png)

# 创建Spring Boot项目模块 

在主项目上点击右键，选择New-Module

![](http://www.wailian.work/images/2018/03/26/5.png)

选择Spring Initializr

![](http://www.wailian.work/images/2018/03/26/6.png)

填写基本信息之后完成模块项目创建

![](http://www.wailian.work/images/2018/03/26/7.png)

采用相同方式再创建一个模块

![](http://www.wailian.work/images/2018/03/26/8.png)

# 修改主项目pom文件

将两个子模块pom中的spring boot父依赖剪切到主项目中的pom文件中

![](http://www.wailian.work/images/2018/03/26/9.png)

在两个子模块pom中添加对父项目的依赖

![](http://www.wailian.work/images/2018/03/26/10.png)

在主项目pom中添加所有的子模块

![](http://www.wailian.work/images/2018/03/26/11.png)

# 测试依赖

utils中添加方法，并在api模块中引入依赖，即可使用

# 打包

注意：多模块项目仅仅需要在启动类所在的模块添加打包插件即可！！不要在父类添加打包插件

删除启动类以外模块中pom文件的build插件,然后在主目录下执行`mvn clean package`即可

[代码地址](https://github.com/wangweiye01/maven)
