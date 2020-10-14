---
title: websocket简介与实战
date: 2020-02-17 09:14:19
tags:
---

![marigold-5582848_1280.jpg](https://p.130014.xyz/2020/10/14/marigold-5582848_1280.jpg)

## 什么是websocket

Websocket是一种在单个TCP连接上进行[全双工](https://baike.baidu.com/item/%E5%85%A8%E5%8F%8C%E5%B7%A5/310007?fr=aladdin)通信的协议

## 为什么需要websocket

HTTP协议有一个缺陷：通信只能由客户端发起

这种单向请求的特点导致如果服务端出现连续状态的变化，客户端要想获知就比较麻烦，我们只能使用"轮询"模式

轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。

![bg2017051502.png](http://s1.wailian.download/2020/02/17/bg2017051502.png)

## 特点

1. 建立在TCP协议之上，服务端的实现比较容易
2. 与HTTP协议有良好的兼容。默认端口也是80和443，并且握手阶段采用HTTP协议，因此握手时不容易屏蔽，能通过各种HTTP代理服务器
3. 数据格式比较轻，新能开销小，通信高效
4. 可以发送文本，也可以发送二进制数据
5. 没有同源限制，客户端可以与任意服务器通信
6. 协议标识符是`ws`(如果加密，则为`wss`)，服务器网址就是URL

## 实战

### STOMP协议

STOMP即Simple Text Orientated Messaging Protocol，简单文本定向消息协议，它提供了一个可互操作的连接格式，允许STOMP客户端与任意STOMP消息代理（Broker）进行交互

### 后端

``` java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 表示客户端订阅地址的前缀信息，也就是客户端接收服务端消息的地址的前缀信息
        config.enableSimpleBroker("/topic", "/user");

        //指服务端接收地址的前缀，意思就是说客户端给服务端发消息的地址的前缀
        config.setApplicationDestinationPrefixes("/app");
        config.setUserDestinationPrefix("/user/");
    }

    // 注册STOMP端点
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/my-websocket").setAllowedOrigins("*").withSockJS();
    }
}
```

```java
@Controller
@EnableScheduling
@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @Autowired
    private SimpMessagingTemplate messagingTemplate;

    @GetMapping("/")
    public String index() {
        return "index";
    }

    @Scheduled(fixedRate = 1000)
    public Object time() throws Exception {
        // 发现消息
        DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        messagingTemplate.convertAndSend("/topic/time", df.format(new Date()));
        return "time";
    }

    @Scheduled(fixedRate = 2000)
    @SendToUser("/greetings")
    public String greeting2() {
        messagingTemplate.convertAndSendToUser("2", "/greetings", "欢迎您，用户: 2");
        return "OK";
    }

    @Scheduled(fixedRate = 2000)
    @SendToUser("/greetings")
    public String greeting1() {
        messagingTemplate.convertAndSendToUser("1", "/greetings", "欢迎您，用户: 1");
        return "OK";
    }

    @Scheduled(fixedRate = 9000)
    public Object notification() {
        // 发送消息
        messagingTemplate.convertAndSend("/topic/notification", "hello world!");
        return "ok";
    }
}
```

### 前端

demo1和demo2都是订阅了两个topic。1个是获得服务器推送的时间，另一个是获得定向推送的消息(demo1向userId=1的用户推送，demo2向userId=2的用户推送)

```javascript
<template>
  <div id="app">
    <div>
      <label>WebSocket连接状态:</label>
      <button type="button" :disabled="connected" @click="connect()">连接</button>
      <button type="button" @click="disconnect()" :disabled="!connected">断开</button>
    </div>

    <div v-if="connected">
      <label>当前服务器时间：{{ time }}</label>
      <br />消息列表：
      <br />
      <hr />
      <table>
        <thead>
          <tr>
            <th>内容</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in lala">
            <td>{{row}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
export default {
  name: "Demo1",
  data() {
    return {
      stompClient: "",
      // 连接状态
      connected: false,
      lala: [],
      time: ""
    };
  },
  methods: {
    connect() {
      const socket = new SockJS("http://localhost:8080/my-websocket");
      this.stompClient = Stomp.over(socket);
      const that = this
      this.stompClient.connect({}, function(frame) {
        // 注册发送消息(demo1和demo2区别在于此)
        that.stompClient.subscribe("/user/1/greetings", function(msg) {
          that.lala.push(msg);
        });
        // 注册推送时间回调
        that.stompClient.subscribe("/topic/time", function(response) {
          that.time = response.body;
        });

        that.connected = true;
      });
    },
    disconnect() {
      if (this.stompClient != null) {
        this.stompClient.disconnect();
      }

      this.connected = false;
      this.lala = [];
    }
  }
};
</script>
```

demo3是订阅后端topic，每隔一段时间推送消息给浏览器，随后浏览器显示[Notification](https://developer.mozilla.org/zh-CN/docs/Web/API/notification)

```javascript
<template>
  <div id="app">
    <div>服务器推送，客户端显示Notification</div>
    <div>WebSocket连接状态:{{ connected }}</div>
    <div>
      <button type="button" :disabled="connected" @click="connect()">连接</button>
      <button type="button" @click="disconnect()" :disabled="!connected">断开</button>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: "Demo3",
  data() {
    return {
      stompClient: "",
      // 连接状态
      connected: false
    };
  },
  methods: {
    connect() {
      var socket = new SockJS("http://localhost:8080/my-websocket");
      this.stompClient = Stomp.over(socket);
      let that = this
      this.stompClient.connect({}, function(frame) {
        // 注册推送时间回调
        that.stompClient.subscribe("/topic/notification", function(response) {
          that.connected = true;
          // 弹窗
          if (window.Notification) {
            var popNotice = function() {
              if (Notification.permission == "granted") {
                var notification = new Notification("Hi，你好", {
                  body: response.body,
                  icon: "https://pcoss.guan18.com/%E6%A9%98%E8%92%9C.jpeg"
                });

                notification.onclick = function() {
                  notification.close();
                };
              }
            };

            if (Notification.permission == "granted") {
              // 已经授权接受通知
              popNotice();
            } else if (Notification.permission != "denied") {
              // 未拒绝接受通知，提示用户授权
              Notification.requestPermission(function(permission) {
                popNotice();
              });
            }
          } else {
            alert("浏览器不支持Notification");
          }
        });

        this.connected = true;
      });
    },
    disconnect() {
      if (this.stompClient != null) {
        this.stompClient.disconnect();
      }

      this.connected = false;
    }
  }
};
</script>
```

demo3演示
![Kapture-2020-02-20-at-14.13.37.gif](http://s1.wailian.download/2020/02/20/Kapture-2020-02-20-at-14.13.37.gif)

[前端源码](https://gitee.com/gttx/websocket)

[后端源码](https://gitee.com/gttx/websocket-api)