---
title: 强大的docker
date: 2018-04-13 21:06:56
tags:
top: 110
---

![](http://www.wailian.work/images/2018/04/18/1211be9c.jpg)

# 什么是Docker

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口 --[百度百科](https://baike.baidu.com/item/Docker/13344470?fr=aladdin)

Docker的思想来自于集装箱，集装箱解决了什么问题？在一艘大船上，可以把货物规整的摆放起来。并且各种各样的货物被集装箱标准化了，集装箱和集装箱之间不会互相影响。那么我就不需要专门运送水果的船和专门运送化学品的船了。只要这些货物在集装箱里封装的好好的，那我就可以用一艘大船把他们都运走。

docker就是类似的理念。现在都流行云计算了，云计算就好比大货轮。docker就是集装箱 --[知乎](https://www.zhihu.com/question/28300645/answer/67707287)

# 镜像

Docker 镜像是用于创建 Docker 容器的只读模板

# 容器

容器是独立运行的一个或一组应用
Docker容器通过Docker镜像来创建

# Docker在不同操作系统下的安装

## Mac安装

Mac下安装Docker很简单，可视化操作即可  [dmg包下载地址](https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac)

查看docker版本

```
docker -v
```

## CentOS安装

```
yum install docker
```

安装完成后，查看docker版本

```
docker version
```

# Docker镜像中心

- [Docker官方仓库](https://hub.docker.com) 国内速度较慢
- [网易云仓库](https://c.163yun.com) 需要注册然后使用，速度较快

# 动手构建Spring Boot+Docker应用

## 通过Spring Initializr创建普通Spring Boot应用

```
@SpringBootApplication
@RestController
public class DockerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DockerApplication.class, args);
    }

    @RequestMapping("/")
    public String home() {
        return "Hello Spring Boot, Docker and CloudComb!";
    }
}
```

## 打包并运行jar包

```
mvn clean package
```

![](http://www.wailian.work/images/2018/04/13/dabao.png)

```
java -jar docker-0.0.1-SNAPSHOT.jar
```

浏览器访问`http://127.0.0.1:8080`输出正常则证明成功

# 容器化构建及运行

## 编写Dockerfile文件

在项目根目录下创建`Dockerfile`文件，

```
FROM hub.c.163.com/bingohuang/jdk8:latest

MAINTAINER wangweiye wwyknight@163.com

COPY target/docker-0.0.1-SNAPSHOT.jar app.jar

ENTRYPOINT ["java","-jar","/app.jar"]
```

此Dockerfile核心功能就是将可执行文件拷贝到镜像中，并在容器启动时默认执行启动命令`java -jar /app.jar`

## docker创建

```
docker build -t spring-boot:latest .
```

`-t`为docker设置镜像名称和版本

详情查看[docker build](https://docs.docker.com/engine/reference/commandline/build/)

## docker运行

```
docker run -d -p 9999:8080 spring-boot
```

`-d` Run container in background and print container ID(在后台运行容器并打印容器ID)

`-p` 设置本机端口和容器端口的映射关系

详情查看[docker run](https://docs.docker.com/engine/reference/commandline/run/)

## 访问

浏览器访问`http://127.0.0.1:8080`输出正常则证明成功

## 推送镜像到网易蜂巢

首先需要一个网易蜂巢账号,[注册地址](https://www.163yun.com/?h=fc)

在命令行中登录蜂巢仓库

```
docker login hub.c.163.com
Username: wwyknight@163.com
Password:
Login Succeeded
```

[推送本地镜像](https://www.163yun.com/help/documents/15587826830438400)

```
docker tag {镜像名或ID} hub.c.163.com/{你的用户名}/{标签名}
```

你的网易云镜像仓库推送地址为`hub.c.163.com/{你的用户名}/{标签名}`
Attention: 此处为你的用户名，不是你的邮箱帐号或者手机号码 登录网易云控制台，页面右上角头像右侧即为「用户名」
推送至不存在的镜像仓库时，自动创建镜像仓库并保存新推送的镜像版本；
推送至已存在的镜像仓库时，在该镜像仓库中保存新推送的版本，当版本号相同时覆盖原有镜像。

```
docker push hub.c.163.com/{你的用户名}/{标签名}
```
默认为私有镜像仓库，推送成功后即可在控制台的「镜像仓库」查看。

![](http://www.wailian.work/images/2018/04/13/12.png)
