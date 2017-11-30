---
title: Spring Boot整合activemq
date: 2017-11-29 12:34:03
tags:
---
# activemq安装

## 下载

[点击下载](http://activemq.apache.org/)

## 解压启动

```
tar -zxvf apache-activemq-5.14.0-bin.tar.gz
```
进去bin目录 cd apache-activemq-5.14.0/bin 启动 ./activemq start

## 打开web管理页面

访问http://IP:8161/admin

启动后，activeMQ会占用两个端口，一个是负责接收发送消息的tcp端口:61616，一个是基于web负责用户界面化管理的端口:8161。这两个端口可以在conf下面的xml中找到。http服务器使用了jetty

# 项目集成activemq

## 配置文件设置application.properties

```
spring.activemq.broker-url=tcp://localhost:61616 
spring.activemq.user=admin 
spring.activemq.password=admin 
spring.activemq.in-memory=true 
spring.activemq.pool.enabled=false
```

## 生产者
```
@Component 
public class Producer implements CommandLineRunner { 
    @Autowired 
        private JmsMessagingTemplate jmsMessagingTemplate; 
    @Autowired 
        private Queue queue; 
    @Override public void run(String... args) throws Exception { 
        send("Sample message"); 
        System.out.println("Message was sent to the Queue"); 
    } 

    public void send(String msg) { 
        this.jmsMessagingTemplate.convertAndSend(this.queue, msg); 
    } 
}
```

## 消费者

```
@Component
public class Consumer { 
    @JmsListener(destination = "sample.queue") 
        public void receiveQueue(String text) { 
            System.out.println(text); 
        } 
}
```

## 程序入口

```
@SpringBootApplication 
@EnableJms 
public class SampleActiveMQApplication { 
    @Bean 
        public Queue queue() { 
            return new ActiveMQQueue("sample.queue"); 
        } 
    public static void main(String[] args) { 
        SpringApplication.run(SampleActiveMQApplication.class, args); 
    } 
}
```

> 代码参考[github地址](https://github.com/spring-projects/spring-boot)
