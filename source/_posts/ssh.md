---
title: CentOS7修改ssh端口号,增强服务器安全
date: 2017-11-27 10:37:06
tags:
---
> 最近发现购买的vps总是有很多次失败尝试登录的记录,感觉很不安全,遂想到修改ssh默认端口来降低风险

# 修改配置文件

```
vim /etc/ssh/sshd_config
```

在**#Port 22**下增加端口**Port 10001**,去掉22端口的注释(以防修改端口失败,默认端口也不能登录)

SSH默认监听端口就是22,上面我保留了22端口,以防修改端口失败,默认端口也不能登录

增加10001端口，大家修改端口时候最好挑10000~65535之间的端口号，10000以下容易被系统或一些特殊软件占用，或是以后新应用准备占用该端口的时候，却被你先占用了，导致软件无法运行。

# 修改SELinux配置

## 查看是否开启了SELinux

```
    sestatus -v
```

如果没有开启,则跳过这一步

## 查看SELinux开放给ssh使用的端口

``` bash
    semanage port -l|grep ssh
```

**ssh_port_t         tcp      22**

## SELinux开放10001端口给ssh

```
    semanage port -a -t ssh_port_t -p tcp 10001
```

完成后再次查看

```
    semanage port -l|grep ssh
```

**ssh_port_t         tcp      22，10001**

# 防火墙放开10001端口

## 修改防火墙配置文件

```
vim /etc/firewalld/zones/public.xml
```

增加100001的端口放开

```
<port protocol="tcp" port="10001"/>
```

# 重启

``` bash
    systemctl restart sshd   重启ssh
    systemctl restart firewalld.service    重启防火墙
    shutdown -r now  重启服务器
```

# 重新登录

``` bash
    ssh root@localhost -p 10001
```

# 如果登录成功注释掉上边的22端口配置

修改了默认端口之后尝试登录的攻击显著降低
