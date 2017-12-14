---
title: 逻辑备份mysql数据库
date: 2017-12-13 15:51:47
tags:
---
# 数据安全高于一切

有时我们为了节省开支，并不会购买云数据库而是选择自建数据库，这时数据安全就极为重要。数据备份是保证安全最有效的方式

# 编写数据备份脚本

```
# /bin/bash
DB_NAME="exchange"
DB_USER="root"
DB_PASSWORD="abc123"
BIN_DIR="/usr/bin"
BACK_DIR="/root/data"
DATE="mysql-`date +'%Y%m%d-%H:%M:%S'`"
LogFile="$BACK_DIR"/dbbakup.log
BackNewFile=$DATE.sql

$BIN_DIR/mysqldump -u$DB_USER -p$DB_PASSWORD $DB_NAME > $BACK_DIR/$DATE.sql

echo -----------------"$(date +"%y-%m-%d %H:%M:%S")"------------------ >> $LogFile

echo  createFile:"$BackNewFile" >> $LogFile

find "/root/data/" -ctime +0 -type f -name "*.sql" -print > deleted.txt

echo -e "delete files:\n" >> $LogFile

cat deleted.txt | while read LINE
do
    rm -rf $LINE
    echo $LINE>> $LogFile
done

echo "---------------------------------------------------------------" >> $LogFile
```

# 利用cron定时执行

利用cron服务定时执行数据备份脚本。该脚本会自动删除过期的sql文件
例如 每天12:50定时执行mysqlback.sh脚本：
```
50 12 * * * /root/mysqlback.sh
```
