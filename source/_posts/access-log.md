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
server.tomcat.accesslog.buffered=true # 缓存日志定期刷新输出（建议设置为true，否则当有请求立即打印日志对服务的响应会有影响）
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

## pattern的配置：

- `%a` - 远程ip地址，注意不一定是原始ip地址，中间可能经过nginx等的转发 

- `%A` - 本地ip

- `%b` - 发送的字节数，不包括HTTP标头，或者如果没有字节发送则使用' - '

- `%B` - 发送的字节数，不包括HTTP标头

- `%h` - 远程主机名（或IP地址，如果连接器的enableLookups为false）

- `%H` - 请求协议

- `%l` - Remote logical username from identd (always returns '-')

- `%m` - 请求方法（GET，POST)

- `%p` - 接受请求的本地端口

- `%q` - 查询字符串（如果存在则用'?'作为前缀，否则为空字符串）

- `%r` - HTTP请求的第一行（包括请求方法，请求的URI）

- `%s` - HTTP的响应代码，如：200,404

- `%S` - User session ID

- `%t` - 日期和时间，Common Log Format格式

- `%u` - 被认证的远程用户

- `%U` - Requested URL path

- `%v` - Local server name

- `%D` - Time taken to process the request, in millis

- `%T` - Time taken to process the request, in seconds

- `%I` - 当前请求的线程名，可以和打印的log对比查找问题

Access log 也支持将cookie、header、session或者其他在ServletRequest中的对象信息打印到日志中，其配置遵循Apache配置的格式（{xxx}指值的名称）：

- `%{xxx}i`  for incoming headers，request header信息
- `%{xxx}o`  for outgoing response headers，response header信息
- `%{xxx}c`  for a specific cookie
- `%{xxx}r`  xxx is an attribute in the ServletRequest
- `%{xxx}s`  xxx is an attribute in the HttpSession
- `%{xxx}t`  xxx is an enhanced SimpleDateFormat pattern (see Configuration Reference document for details on supported time patterns)

# 内置模板

`server.tomcat.accesslog.pattern`中内置了两个日志格式模板，分别是common和combined

- `common`: `%h %l %u %t "%r" %s %b`

- `combined`: `%h %l %u %t "%r" %s %b "%{Referer}i" "%{User-Agent}i"`


参考[Tomcat access log配置](http://www.cnblogs.com/chrischennx)
