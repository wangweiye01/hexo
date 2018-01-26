---
title: MySql主从配置
date: 2018-01-26 09:07:07
tags:
---
![pic](http://www.wailian.work/images/2018/01/26/5d274a04c82163baf3c74ae5c97d16e80a641ed24163c7-YnTeKa_fw658.jpg)

mysql版本使用的是5.7.20
主库的IP为:47.104.71.88，从库的IP为:45.77.13.74

# 读写分离(主从库)

原理：让主库(master)处理事务性增改删，而从库(slave)处理查询操作

# 主库创建用户

```
GRANT REPLICATION SLAVE,RELOAD,SUPER ON *.* TO synchrouser@45.77.13.74 IDENTIFIED BY 'w123456W!';
```

在主库创建一个synchrouser用户密码为w123456W!，并允许从库以synchrouser用户来登录

# 配置主库

在[mysqld]下增加配置

```
server-id=88
log_bin=mysql-bin
binlog_format=mixed
```

server-id在数据库配置中必须唯一，一般为IP最后一个节点（例如：47.104.71.88，则设置为88）设置完成后，重启mysql

# 配置从库

在[mysqld]增加配置

```
server-id=74
```

设置完成后，重启mysql

在主库执行:`show master status;`

![](http://www.wailian.work/images/2018/01/26/WX20180126-091715.png)

根据以上主库的信息设置从库

```
change master to master_host='47.104.71.88',master_user='synchrouser',master_password='w123456W!',master_log_file='mysql-bin.000005',master_log_pos=840;
```

master_log_file字段对应了主库的File，master_log_pos字段对应了主库的Position

# 启动主从同步

从库执行

```
start slave;
```

# 检查是否配置成功

![](http://www.wailian.work/images/2018/01/26/4.png)

如果Slave_IO_Running和Slave_SQL_Running都为Yes，代表配置成功

# 测试主从同步

主库创建一个库，一个表，观察从库是否同样创建

# 附录：

到这里，全部库的主从配置就完成了，实际应用中可能会用到单个表的同步，或者部分表的同步，只需要在主库的/etc/my.cnf里加上

只复制某个表replicate-do-table=tablename
只复制某些表（可用匹配符）replicate-wild-do-table=tablename%
只复制某个库replicate-do-db=dbname
只复制某些库replicte-wild-do-db=dbname%
不复制某个表replicate-ignore-table=tablename
