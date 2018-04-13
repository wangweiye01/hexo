---
title: Spring Boot内嵌Tomcat临时目录问题 
date: 2018-04-12 17:22:59
tags:
top: 10
---

> 最近发现线上一个项目日志突然报错，最终找到解决方法记录一下

# 错误信息

`java.io.IOException: The temporary upload location [/tmp/tomcat.948083514929465118.8889/work/Tomcat/localhost/ROOT] is not valid`

![](http://www.wailian.work/images/2018/04/12/21.png)

# 原因

参考 https://github.com/spring-projects/spring-boot/issues/5009

`tmpwatch` – removes  files  which haven’t been accessed for a period of time

如上所言，删除指定的目录中一段时间未访问的文件。一般对于/tmp下的文件或日志文件

意思是tomcat的临时目录会被`tmpwatch`删除掉，甚至可能删除掉`class`文件，导致错误的发生

# 解决方法

## 1.启动时指定新的临时目录

```
-Djava.io.tmpdir=/var/tmp
```

## 2.配置文件中设置新的临时目录

```
server:
    tomcat:
       basedir: /var/tmp/
```

## 3.代码中配置tomcat 临时目录

```
@Configuration
public class MultipartConfig {
    @Bean
    MultipartConfigElement multipartConfigElement() {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        String location = System.getProperty("user.dir") + "/data/tmp";
        File tmpFile = new File(location);
        if (!tmpFile.exists()) {
            tmpFile.mkdirs();
        }
        factory.setLocation(location);
        return factory.createMultipartConfig();
    }
}
```

[参考链接](http://www.wtnull.com/v/SPRING/Spring%20boot%20%E5%86%85%E5%B5%8Ctomcat%E4%B8%B4%E6%97%B6%E7%9B%AE%E5%BD%95%E4%B8%8D%E5%AD%98%E5%9C%A8%20%E9%94%99%E8%AF%AF.html)
