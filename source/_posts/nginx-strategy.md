---
title: nginx负载均衡策略
date: 2017-11-29 08:50:05
tags:
---
# 轮询（默认策略）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除

# 指定权重

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
```
upstream servername { 
    server 192.168.0.1 weight=1; 
    server 192.168.0.2 weight=12; 
} 
```

# IP绑定 ip_hash
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
```
upstream backserver { 
    ip_hash; 
    server 192.168.0.14:88; 
    server 192.168.0.15:80; 
} 
```

# fair（第三方）

按后端服务器的响应时间来分配请求，响应时间短的优先分配。
```
upstream backserver { 
    server server1; 
    server server2; 
    fair; 
} 
```
这种策略具有很强的自适应性，但是实际的网络环境往往不是那么简单，因此须慎用。

# url_hash（第三方）

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
```
upstream backserver { 
    server squid1:3128; 
    server squid2:3128; 
    hash $request_uri; 
    hash_method crc32; 
} 
```
