---
title: Spring Boot配置access log 
date: 2018-04-13 10:49:14
tags:
top: 80
---

> Spring Boot集成logback可以解决大部分日志需求，但是缺少访问日志，这在生产环境也是一种隐患。配合Tomcat Access Log可以完美解决这个问题

# 配置

在Spring boot中使用了内嵌的tomcat，可以通过`server.tomcat.accesslog`配置tomcat 的access日志

默认日志如下:

```
server.tomcat.accesslog.buffered=true # 缓存日志定期刷新输出
server.tomcat.accesslog.directory=logs # 日志文件路径，可以是相对于tomcat的路径也可是绝对路径 
server.tomcat.accesslog.enabled=false # 是否开启访问日志
server.tomcat.accesslog.file-date-format=.yyyy-MM-dd # 放在日志文件名中的日期格式 
server.tomcat.accesslog.pattern=common # 日志格式，在下面详解 
server.tomcat.accesslog.prefix=access_log # 日志文件名前缀
server.tomcat.accesslog.rename-on-rotate=false # 推迟在文件名中加入日期标记，直到日志分割时 
server.tomcat.accesslog.request-attributes-enabled=false # 为请求使用的IP地址，主机名，协议和端口设置请求属性 
server.tomcat.accesslog.rotate=true # 是否启用访问日志分割
server.tomcat.accesslog.suffix=.log # 日志名后缀
```

`server.tomcat.accesslog.pattern`中内置了两个日志格式模板，分别是common和combined

- `common`: `%h %l %u %t "%r" %s %b`

- `combined`: `%h %l %u %t "%r" %s %b "%{Referer}i" "%{User-Agent}"`
