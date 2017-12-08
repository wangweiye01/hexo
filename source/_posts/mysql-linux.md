---
title: linux导入导出mysql数据
date: 2017-12-08 13:55:18
tags:
---
## linux导入sql

```
mysql -h [host] -u [userName] -p [databaseName] < [data].sql
```

## linux导出库

```
mysqldump -h [host] -u [username] -p --databases [databasename] > [fileName].sql
```

## linux导出单表

```
mysqldump -h [host] -u [username] -p [dabaseName] [tableName] > [fileName].sql
```
