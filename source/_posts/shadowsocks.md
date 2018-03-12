---
title: vps搭建shadowsocks
date: 2017-11-28 10:18:55
tags:
---
![pic](http://s1.wailian.download/2018/01/18/3bf44e943f8d77713982edd8c8e0fa478c6bbf622633d-zCwRVd_fw658.jpg)
# 购买vultr服务

注册vultr,购买服务(建议地址选Tokyo,操作系统选择CentOS,配置选择最小配置),然后部署服务器

# 安装shadowsocks

> 本文安装是是使用CentOS7系统

## 连接服务器

## 依次执行以下命令

``` bash
wget –no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

## 等几秒钟,根据提示输入shadowsocks密码及端口号

![安装ss](http://img.blog.csdn.net/20171110091659947?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMTY3MTI2NzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 成功后出现以下页面,妥善保存信息

![安装成功后](http://img.blog.csdn.net/20171110091341666?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMTY3MTI2NzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 配置维护Shadowsocks

## 配置多用户多端口

修改/etc/shadowsock.json文件

```
{
    "server":"my_server_ip"，
        "local_address": "127.0.0.1",
        "local_port":1080,
        "port_password": {
            "8381": "foobar1",
            "8382": "foobar2",
            "8383": "foobar3",
            "8384": "foobar4"
        },
        "timeout":300,
        "method":"aes-256-cfb",
        "fast_open": false
}
```

其中，port_password就是将之前的服务器端口和对应的密码结合起来。然后通过使用以下命令，就可以启动多端口多用户了。

```
启动：systemctl start shadowsocks.service
停止：systemctl stop shadowsocks.service
重启：systemctl restart shadowsocks.service
状态：systemctl status shadowsocks.service
```

centos默认的防火墙机制，会阻隔掉我们的多端口配置。所以，解决方法就是，将这个端口打开tcp和udp通信。这里需要说明的事，在centos版本的更新迭代过程中，centos7和centos6之间的差异性较大。在centos7，采用的是最新的防火墙filewall而不是传统的iptables。
具体操作为，先进入firewalled的配置端口目录，路径为etc/firewalled/zones打开public.xml文件进行端口的编辑.例如

```
<port protocol="tcp" port="449"/>   
<portprotocol="udp" port="443"/>
```

即添加了tcp和udp的权限.然后systemctl restart firewalld.service 重启防火墙，端口就添加到防火墙的白名单中啦。

# 下载(Shadowsocks客户端)

## mac和windows

访问

```
https://github.com/shadowsocks
```

![client](http://img.blog.csdn.net/20171110093052673?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMTY3MTI2NzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
*Shadowsocks-windows为windows客户端*
*ShadowsocksX-NG为mac客户端*

## iphone
商店搜索super wingy
![super_wingy](http://img.blog.csdn.net/20171110092642061?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMTY3MTI2NzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# KCPTun加速

搭建好了Shadowsocks相当于搭建成功了梯子，但是梯子太长，即时梯子带宽足够宽，线路质量也是不忍直视，此时就需要KCPTun了。

KCPTun是一个使用可信UDP来加速TCP传输速度的网络软件。

KCP 是一个快速可靠协议，能以比 TCP浪费10%-20%的带宽的代价，换取平均延迟降低 30%-40%，且最大延迟降低三倍的传输效果。

## KCPTun服务安装

```
// 下载脚本
wget https://raw.githubusercontent.com/kuoruan/kcptun_installer/master/kcptun.sh
// 赋予权限
chmod +x ./kcptun.sh
// 执行脚本
./kcptun.sh
```

# vultr购买推荐地址

```
https://www.vultr.com/?ref=7259520
```
