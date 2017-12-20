---
title: Linux用户权限
date: 2017-12-20 16:10:19
tags:
---
CentOS默认允许任何用户通过ssh登入，但是在企业应用中为了保证服务器的安全性一般不允许用户直接通过root用户登录，而是通过普通用户登录然后切换到root或者使用sudo权限来执行root用户命令

# 创建用户

## 创建普通用户

```
useradd wangweiye
```
## 为普通用户设置密码

```
passwd wangweiye
```

## 将普通用户移入wheel用户组

```
usermod -G wheel wangweiye
```

> 在Linux中wheel组类似于管理员组

这样就可以使用普通用户wangweiye来进行登录了,而且该用户拥有sudo权限

# 禁止root用户ssh登录

## 修改配置文件

修改`/etc/ssh/sshd_config`将`PermitRootLogin`设置为no即可

## 重启ssh服务

`systemctl restart sshd.service`重启ssh服务
