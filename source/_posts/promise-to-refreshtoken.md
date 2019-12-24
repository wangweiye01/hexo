---
title: axios使用Promise刷新token
date: 2019-12-24 09:18:50
tags: 前端,vue
---
![](http://s1.wailian.download/2019/12/24/snow-ball-4708528_1280.jpg)

# 问题

最近遇到个需求：前端登录后，后端返回`token`和`refreshToken`，当`token`过期时需要使用`refreshToken`去获取新的`token`和新的`refreshToken`，前端需要做到无感知刷新`token`

# 方法

利用 axios 的拦截器，拦截返回后的数据。当接口返回`token`过期后，先刷新`token`然后重试

# 难点

由于发起网络请求是异步的，当同时发起多个请求，而刷新 token 的接口还没有返回时，如何让这些请求等待刷新接口的返回，然后使用返回的新 token 重新发起请求呢？

# 实现

```javascript
import axios from "axios";

// 创建一个axios实例
const instance = axios.create({
  baseURL: "/api",
  timeout: 300000,
  headers: {
    "Content-Type": "application/json",
    "X-Token": getLocalToken() // headers塞token
  }
});

// 从localStorage中获取token
function getLocalToken() {
  const token = window.localStorage.getItem("token");
  return token;
}

// 给实例添加一个setToken方法，用于登录后将最新token动态添加到header，同时将token保存在localStorage中
instance.setToken = token => {
  instance.defaults.headers["X-Token"] = token;
  window.localStorage.setItem("token", token);
};

function refreshToken() {
  // instance是当前request.js中已创建的axios实例
  return instance.post("/refreshtoken").then(res => res.data);
}

// 是否正在刷新的标记
let isRefreshing = false;
// 重试队列，每一项将是一个待执行的函数形式
let retryRequests = [];
// 请求后拦截 axios.interceptors.request.use()
instance.interceptors.response.use(
  response => {
    const { code } = response.data;
    // 约定当code === 4001时，为token过期
    if (code === 4001) {
      // config是为请求提供的配置信息
      const config = response.config;
      if (!isRefreshing) {
        isRefreshing = true;
        return refreshToken().then(res => {
            const { token } = res.data;
            instance.setToken(token);
            config.headers["X-Token"] = token;
            // 注意： 原请求已经将baseURL进行拼接，此处不要重复拼接
            config.baseURL = "";
            // 将队列中的请求进行重试
            retryRequests.forEach(cb => cb(token));
            retryRequests = [];
            return instance(config);
          }).catch(res => {
            console.error("refreshtoken error =>", res);
            window.location.href = "/";
          }).finally(() => {
            // 保证下次刷新能够正常进入
            isRefreshing = false;
          });
      } else {
        // 正在刷新token，将返回一个未执行resolve的promise
        return new Promise(resolve => {
          // 将resolve放进队列，用一个函数形式来保存，等token刷新后直接执行
          retryRequests.push(token => {
            config.baseURL = "";
            config.headers["X-Token"] = token;
            resolve(instance(config));
          });
        });
      }
    }
    return response;
  },
  error => {
    return Promise.reject(error);
  }
);

export default instance;
```
