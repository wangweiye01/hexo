---
title: access_token高并发情况下被覆盖
date: 2018-01-22 11:25:20
tags:
---

# 问题描述

最近搞微信公众平台接口时，我们获得access_token的处理方式是：先在缓存中查找，如果找到，返回；否则，调用微信接口获得，放入缓存（有时效），然后返回；

此处的逻辑并不严谨

如果有两个线程  **同时**  获得access_token，而且此时恰好access_token在缓存中因为过期而删除了，那么两个线程都会调用微信接口请求access_token，这时就会出现第二次请求的结果把第一次的结果覆盖掉的情况。

最初实现方法：

```
public JSONObject getAccessToken(String appid, String appsecret) {
    String reqestUrl = ACCESS_TOKEN_URL.replace("APPID", appid).replace("APPSECRET", appsecret);

    RedisUtils<String> redisUtils = new RedisUtils<>(redisTemplate);

    String accessToken = redisUtils.get(ACCESS_TOKEN_CACHE_KEY);

    if (StrUtil.isNotEmpty(accessToken)) {
        return JSONObject.parseObject(accessToken);
    }

    JSONObject resultJSON = JSONObject.parseObject(HttpUtil.get(reqestUrl));

    if (StrUtil.isNotEmpty(resultJSON.getString("errcode"))) {
        log.info(resultJSON.getString("errmsg"));
        return null;
    }

    redisUtils.set(ACCESS_TOKEN_CACHE_KEY, resultJSON.toJSONString(), 7000L);

    return resultJSON;
}
```

# 解决方法

把请求微信接口的方法加入同步代码块中

```
public JSONObject getAccessToken(String appid, String appsecret) {
    String reqestUrl = ACCESS_TOKEN_URL.replace("APPID", appid).replace("APPSECRET", appsecret);

    RedisUtils<String> redisUtils = new RedisUtils<>(redisTemplate);

    String accessToken = redisUtils.get(ACCESS_TOKEN_CACHE_KEY);

    if (StrUtil.isNotEmpty(accessToken)) {
        return JSONObject.parseObject(accessToken);
    }

    // 防止高并发情况下，access_token覆盖的问题
    synchronized (appid) {
        // 再次验证缓存中的数据
        String accessToken2 = redisUtils.get(ACCESS_TOKEN_CACHE_KEY);
        if (StrUtil.isNotEmpty(accessToken2)) {
            log.info("multi thread hits!");
            return JSONObject.parseObject(accessToken2);
        }

        JSONObject resultJSON = JSONObject.parseObject(HttpUtil.get(reqestUrl));

        if (StrUtil.isNotEmpty(resultJSON.getString("errcode"))) {
            log.info(resultJSON.getString("errmsg"));
            return null;
        }

        redisUtils.set(ACCESS_TOKEN_CACHE_KEY, resultJSON.toJSONString(), 7000L);

        return resultJSON;
    }
}
```
