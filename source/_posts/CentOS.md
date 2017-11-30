---
title: CentOS7安装好服务,外网不能访问
date: 2017-11-28 16:00:00
tags:
---
# 问题:CentOS安装nginx后,通过外网不能访问

# 怀疑:nginx使用80端口,被防火墙拦截

# 解决办法

## 编辑防火墙配置文件

```
vim /etc/firewalld/zones/public.xml
```

## 增加80端口的开放

```
<port protocol="tcp" port="80"/>
```

## 重启防火墙

```
systemctl restart firewalld.service
```
