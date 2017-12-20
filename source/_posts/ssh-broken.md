---
title: 避免ssh超时断开
date: 2017-12-20 09:24:21
tags:
---
用ssh连接服务器经常遇到长时间不操作而被服务器踢出的情况，提示如下
> Write failed: Broken pipe

# 方案一:客户端配置

在`/etc/ssh/ssh_config`中添加配置
```
ServerAliveInterval 60
```
之后，当使用ssh时每隔60s客户端都会向服务端发送一个KeepAlive请求，避免被踢

# 方案二：服务端配置

在`/etc/ssh/sshd_config`中添加配置
```
ClientAliveInterval 60
```
重启服务器后设置生效
注意：这种方式每一个连接服务器的客户端都受到影响，安全性会有一定的下降
