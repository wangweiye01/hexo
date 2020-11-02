---
title: Spring Boot多模块项目创建以及需要注意的问题
date: 2018-03-26 15:06:22
tags:
---
![](http://www.wailian.work/images/2018/03/28/f5462f69549047be1b086629e548eefc8d74eca329bea-IiKAbw_fw658.jpg)

> 比起传统复杂的单体工程，使用Maven的多模块配置，可以帮助项目划分模块，鼓励重用，防止POM变得过于庞大，方便某个模块的构建，而不用每次都构建整个项目，并且使得针对某个模块的特殊控制更为方便

# 构建项目

- 可以使用`Spring Initializr`先创建一个普通的spring boot项目，然后删除`src`目录以及其他无用文件

- 然后修改主项目的pom文件，在其中添加`<packaging>pom</packaging>`

![WX20201102-171326.png](https://p.130014.xyz/2020/11/02/WX20201102-171326.png)

- 接着添加`<dependencyManagement>`用于管理项目中所有依赖的版本信息。当子moudle中需要该依赖时，不需要填写版本号，以此来保证项目中的版本一致性

[![WX20201102-171601.png](https://p.130014.xyz/2020/11/02/WX20201102-171601.png)](http://www.wailian.work/image/QKITi4)

## 创建Spring Boot子模块 

使用`Spring Initializr`创建普通的spring boot模块

- 将`pom`中的`parent`信息改为主项目信息；并修改`<packaging>`为`jar`

[![WX20201102-172112.png](https://p.130014.xyz/2020/11/02/WX20201102-172112.png)](http://www.wailian.work/image/QKIrAk)

- 注意：如果该子模块仍需要被其他模块依赖，则修改打包插件如图，增加`<configuration>`的配置，使该模块默认打包为`普通jar`而非`可执行jar`打包后生成的两个jar包如图所示

![WX20201102-172559.png](https://p.130014.xyz/2020/11/02/WX20201102-172559.png)


## 如何做到多环境配置文件合理覆盖

在子项目中有默认的配置文件，其他依赖于该子项目的项目可以选择性覆盖配置文件中的信息

- 在子项目中创建默认的配置文件

![WX20201102-173211.png](https://p.130014.xyz/2020/11/02/WX20201102-173211.png)

[![WX20201102-173338.png](https://p.130014.xyz/2020/11/02/WX20201102-173338.png)](http://www.wailian.work/image/QKI2jI)

- 在主项目中使用`spring.profiles.include`引入子项目中的配置文件，来覆盖其中的配置信息（具有相同的配置时后面的覆盖前面的 service覆盖common，dev覆盖service）

![WX20201102-173609.png](https://p.130014.xyz/2020/11/02/WX20201102-173609.png)

# 测试

在根目录下执行`mvn clean package`即可打包所有的子项目

[代码地址](https://gitee.com/doublew/sha)
