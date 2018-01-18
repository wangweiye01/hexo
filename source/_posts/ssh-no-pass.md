---
title: ssh免密登录
date: 2017-12-18 20:45:29
tags:
---
![pic](http://s1.wailian.download/2018/01/18/77e75ad8e3c3226b53206731154ac0f7a0c27a97a99c5-2wKmIY_fw658.jpg)
# 场景

客户端A想要免密登录服务器B

# 客户端A

## 生成SSH KEY

```
ssh-keygen -t rsa -C "your_email@example.com"
```

## 查看客户端公钥

```
cat ~/.ssh/id_rsa.pub
```

# 服务端B

## 生成SSH KEY

```
ssh-keygen -t rsa -C "your_email@example.com"
```

## 连接A-B

复制A的公钥到B的
```
~/.ssh/authorized_keys
```

# 重新登录
