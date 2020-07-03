---
title: 接口限流的令牌桶算法以及使用方法
date: 2020-07-03 10:11:12
tags:
---

在开发高并发系统时，有三把利器来保护系统：**缓存** **降级** **限流**

## 限流目的

限流的目的是通过对并发请求进行限速，一旦达到限制速率则可以拒绝服务或者排队等待等处理

## 限流算法

常用的限流算法有**令牌桶算法**和**漏桶算法** 

漏桶算法要求处理请求以一个恒定的速率，不能允许突发请求的快速处理，而令牌桶算法就比较适合

令牌桶算法的原理就是系统以恒定的速率产生令牌放入令牌桶中。令牌桶有容量，当满时，再放入的令牌就会被丢弃。当想处理一个请求的时候，需要从令牌桶中取出一个令牌，如果没有，则拒绝（非阻塞式）或者等待（阻塞式）

![4179645397-5b6e4903ec371_articlex.png](http://s1.wailian.download/2020/07/03/4179645397-5b6e4903ec371_articlex.png)

Google开源项目Guava中RateLimiter使用的就是令牌桶算法，下面实例是使用自定义注解实现限制接口流量


定义一个注解

```java
@Inherited
@Documented
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimitAspect {

}
```

定义切面

```java
@Component
@Scope
@Aspect
public class RateLimitAop {
    @Autowired(required = false)
    private HttpServletResponse response;

    private RateLimiter rateLimiter = RateLimiter.create(5); //比如说，我这里设置"并发数"为5

    @Pointcut("@annotation(cc.wangweiye.ratelimit.RateLimitAspect)")
    public void serviceLimit() {

    }

    @Around("serviceLimit()")
    public Object around(ProceedingJoinPoint joinPoint) {
        Boolean flag = rateLimiter.tryAcquire();
        Object obj = "无返回";
        try {
            if (flag) {
                obj = joinPoint.proceed();
            } else {
                String result = "failure";

                output(response, result);
            }
        } catch (Throwable e) {
            e.printStackTrace();
        }
        System.out.println("flag=" + flag + ",obj=" + obj);
        return obj;
    }

    public void output(HttpServletResponse response, String msg) throws IOException {
        response.setContentType("application/json;charset=UTF-8");
        ServletOutputStream outputStream = null;
        try {
            outputStream = response.getOutputStream();
            outputStream.write(msg.getBytes("UTF-8"));
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            outputStream.flush();
            outputStream.close();
        }
    }
}
```

[源码地址](https://github.com/wangweiye01/ratelimit)