---
title: 3.Spring Cloud：服务消费（Ribbon）【Dalston版】
date: 2018-01-08 16:56:06
top: 1
categories: 微服务
---

![](http://www.wailian.work/images/2018/04/11/c823eb1967ed6f4573223516337156c2323dc33f1ce08-d4m6uZ_fw658.jpg)

> 通过上一篇《Spring Cloud：服务消费（基础）》，我们已经学会如何通过LoadBalancerClient接口来获取某个服务的具体实例，并根据实例信息来发起服务接口消费请求。但是这样的做法需要我们手工的去编写服务选取、链接拼接等繁琐的工作，对于开发人员来说非常的不友好。所以，下来我们看看Spring Cloud中针对客户端负载均衡的工具包：Spring Cloud Ribbon

# 动手试一试

下面的例子，我们将利用之前构建的eureka-server作为服务注册中心、eureka-client作为服务提供者作为基础。而基于Spring Cloud Ribbon实现的消费者，我们可以根据eureka-consumer实现的内容进行简单改在就能完成，具体步骤如下：

- 根据eureka-consumer复制一个服务消费者工程，命名为：eureka-consumer-ribbon。在pom.xml中增加下面的依赖：

```
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
    </dependency>
</dependencies>
```

- 修改应用主类。为RestTemplate增加@LoadBalanced注解：

```
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }
}
```

- 修改Controller。去掉原来通过LoadBalancerClient选取实例和拼接URL的步骤，直接通过RestTemplate发起请求。

```
@RestController
public class DcController {
    @Autowired
    RestTemplate restTemplate;
    @GetMapping("/consumer")
    public String dc() {
        return restTemplate.getForObject("http://eureka-client/dc", String.class);
    }
}
```

可以看到这里，我们除了去掉了原来与LoadBalancerClient相关的逻辑之外，对于RestTemplate的使用，我们的第一个url参数有一些特别。这里请求的host位置并没有使用一个具体的IP地址和端口的形式，而是采用了服务名的方式组成。那么这样的请求为什么可以调用成功呢？因为Spring Cloud Ribbon有一个拦截器，它能够在这里进行实际调用的时候，自动的去选取服务实例，并将实际要请求的IP地址和端口替换这里的服务名，从而完成服务接口的调用。

在完成了上面你的代码编写之后，读者可以将eureka-server、eureka-client、eureka-consumer-ribbon都启动起来，然后访问http://localhost:2101/consumer ，来跟踪观察eureka-consumer-ribbon服务是如何消费eureka-client服务的/dc接口的，并且也可以通过启动多个eureka-client服务来观察其负载均衡的效果。

> 转载自[程序猿DD-翟永超](http://blog.didispace.com/)
