---
title: git新建项目推送远程
date: 2018-02-02 13:58:28
tags:
---

1. 在github或者码云上新建仓库

2. git init在本地新建仓库

3. git remote add origin git@git.oschina.net:wangweiye/SpringBoot-Learning.git

   执行以上命令将本地仓库与远程仓库相连

4. git add .

5. git commit -m ""

6. git pull origin master

   可能报错，fatal: refusing to merge unrelated histories

   解决方法：git pull origin master --allow-unrelated-histories

7. git push -u origin master
