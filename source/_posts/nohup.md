---
title: Spring Boot后台启动不打印nohup.out
date: 2017-11-29 11:31:35
tags:
---
```
nohup java -jar yatai_pro-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod >/dev/null &
```
>/dev/null 表示将标准输出信息重定向到”黑洞”
