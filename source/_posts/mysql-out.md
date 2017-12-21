---
title: mysql设置外部访问
date: 2017-12-21 16:45:54
tags:
---
mysql5.7设置允许外网登录数据库

# 创建host

## 进入mysql数据库，修改`user`表

```
use mysql
```

```
update user set host='%' where user='root'
```

## 刷新权限
```
flush privileges
```

# 授权用户

## 任意主机以用户root和密码mypwd连接到mysql数据库

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'  IDENTIFIED BY 'mypwd'  WITH GRANT OPTION
```

## IP为192.168.1.11的主机以用户myuser和密码mypwd连接到mysql数据库

```
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'192.168.1.11' IDENTIFIED BY 'mypwd' WITH GRANT OPTION
```

## 刷新权限
```
flush privileges
```

# 可能遇到的问题

## 设置后仍不能访问：防火墙放开mysql的tcp端口
## 如果是阿里ECS用户：配置安全组规则放开mysql端口
