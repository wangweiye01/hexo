---
title: spring booot多环境配置
date: 2017-12-01 08:49:43
tags:
---
我们在开发Spring Boot应用时，通常同一套程序会被应用和安装到几个不同的环境，比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。


对于多环境的配置，各种项目构建工具或是框架的基本思路是一致的，通过配置多份不同环境的配置文件，再通过打包命令指定需要打包的内容之后进行区分打包，Spring Boot也不例外，或者说更加简单。


在Spring Boot中多环境配置文件名需要满足 `application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识，比如：

```
application-dev.properties：开发环境 
application-test.properties：测试环境 
application-prod.properties：生产环境
```

至于哪个具体的配置文件会被加载，需要在`application.properties`文件中通过`spring.profiles.active`属性来设置，其值对应`{profile}`值。

如：`spring.profiles.active=test`就会加载`application-test.properties`配置文件内容

下面，以不同环境配置不同的服务端口为例，进行样例实验。

针对各环境新建不同的配置文件`application-dev.properties`、`application-test.properties`、`application-prod.properties`

测试不同配置的加载: 
执行```java -jar xxx.jar --spring.profiles.active=prod```也就是生产环境的配置(prod)按照上面的实验，可以如下总结多环境的配置思路：

**application.properties中配置通用内容，并设置spring.profiles.active=dev，以开发环境为默认配置**
**application-{profile}.properties中配置各个环境不同的内容**
**通过命令行方式去激活不同环境的配置**

转载自[程序猿DD-翟永超](http://blog.didispace.com/springbootproperties/)
