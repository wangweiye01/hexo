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
spring.activemq.pool.enabled=false
```

## 生产者
```
@Service
public class Producer {
    @Resource
    private JmsMessagingTemplate jmsMessagingTemplate;

    public void sendMsg(String destName, String message) {
        Destination destination = new ActiveMQQueue(destName);
        jmsMessagingTemplate.convertAndSend(destination, message);
    }
}

```

## 消费者

```
@Service
public class Consumer {
    @JmsListener(destination = "test.queue")
    public void receiveMsg(String text) {
        System.out.println(">>>>->>>收到消息:" + text);
    }
}
```

## 发布者

```
@Service
public class Publisher {
    @Resource
    private JmsMessagingTemplate jmsMessagingTemplate;

    public void publish(String destName, String message) {
        Destination destination = new ActiveMQTopic(destName);

        jmsMessagingTemplate.convertAndSend(destination, message);
    }
}
```

## 订阅者

```
@Service
public class Subscriber {
    @JmsListener(destination = "test.topic", containerFactory = "myJmsContainerFactory")
    public void subscribe(String text) {
        System.out.println("===<<<<收到订阅消息:" + text);
    }

    @Bean
    JmsListenerContainerFactory myJmsContainerFactory(ConnectionFactory connectionFactory) {
        SimpleJmsListenerContainerFactory factory = new SimpleJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setPubSubDomain(true);
        return factory;
    }
}
```

在pub/sub模式中，对消息的监听需要对containerFactory的配置

> 代码参考[github地址](https://github.com/wangweiye01/activemq)
