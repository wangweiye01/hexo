---
title: nginx配置CORS实现跨域
date: 2018-04-10 14:58:37
tags:
top: 99
---

![toutu](http://www.wailian.work/images/2018/04/11/502af7eae3feeb1e4b5c376c2f24e9b206182f1b187f73-7ntUXd_fw658.jpg)

> 单服务器应用不用nginx代理时在Java中都是在Filter中实现的跨域设置，如何在请求到达应用服务器之前实现跨域的设置呢？使用nginx配置实现

# 什么是CORS

CORS是一个W3C标准，全称是跨域资源共享(Cross-origin resource sharing)。它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。

# 实现方法

```
server {
    listen       80;
    server_name  agent.bater.top;

    root /var/www/html/api_agent/public;

    location / {

        if ($request_method = 'OPTIONS') {
            add_header Access-Control-Allow-Origin $http_origin;
            add_header Access-Control-Allow-Credentials true;
            add_header Access-Control-Allow-Methods 'GET,POST,PUT,DELETE,OPTIONS';
            add_header Access-Control-Allow-Headers 'Authorization,X-Requested-With,Content-Type,Origin,Accept,Cookie';
            add_header Access-Control-Max-Age 3600;
            add_header Content-Length 0;
            return 202;
        }

        add_header Access-Control-Allow-Origin $http_origin;
        add_header Access-Control-Allow-Credentials true;
        add_header Access-Control-Allow-Methods 'GET,POST,PUT,DELETE,OPTIONS';
        add_header Access-Control-Allow-Headers 'Authorization,X-Requested-With,Content-Type,Origin,Accept,Cookie';
        
        proxy_pass http://localhost:8080/;
    }
}
```

`Access-Control-Allow-Origin`: 它是W3C标准里用来检查该跨域请求是否可以被通过(Access Control Check)。如果需要跨域，解决方法就是在资源的头中加入Access-Control-Allow-Origin 指定你授权的域。

`Access-Control-Allow-Credentials`: 它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。 

`Access-Control-Allow-Methods`: 它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法

`Access-Control-Allow-Headers`: 它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段

`Access-Control-Max-Age`: 该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是1小时（3600秒），即允许缓存该条回应3600秒，在此期间，不用发出另一条预检请求

`Content-Length`: 用于描述HTTP消息实体的传输长度
