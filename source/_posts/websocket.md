---
title: Spring Boot实现WebSocket消息推送
date: 2018-03-01 17:05:06
tags:
---

![pic](http://www.wailian.work/images/2018/03/02/3b41bb4144a7478ef7d1937c3a0ec56975e163f61a9ba-n8Syo1_fw658.jpg)

# 什么是WebSocket

WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。

# 配置

```
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

	@Override
	public void configureMessageBroker(MessageBrokerRegistry config) {
		config.enableSimpleBroker("/topic");
		config.setApplicationDestinationPrefixes("/app");
	}

	@Override
	public void registerStompEndpoints(StompEndpointRegistry registry) {
		registry.addEndpoint("/my-websocket").withSockJS();
	}
}
```

这里配置了以"/app"开头的websocket请求url和名为"my-websocket"的endpoint

1. @EnableWebSocketMessageBroker注解表示开启使用STOMP协议来传输基于代理的消息
2. registerStompEndpoints方法表示注册STOMP协议的节点，并指定映射的URL
3. `registry.addEndpoint("/my-websocket").withSockJS()`这一行代码用来注册STOMP协议节点，同时指定使用SockJS协议。
4. configureMessageBroker方法用来配置消息代理，由于我们是实现推送功能，这里的消息代理是/topic

# 推送消息类

```
public class SocketMessage {

	public String message;

	public String date;

}
```


# 控制器

```
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

	@MessageMapping("/send")
	@SendTo("/topic/send")
	public SocketMessage send(SocketMessage message) throws Exception {
		DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		message.date = df.format(new Date());
		return message;
	}

	@Scheduled(fixedRate = 1000)
	@SendTo("/topic/callback")
	public Object callback() throws Exception {
		// 发现消息
		DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		messagingTemplate.convertAndSend("/topic/callback", df.format(new Date()));
		return "callback";
	}
}
```

`@MessageMapping`注解和`@RequestMapping`类似，用来发送消息到特定路径

`@SendTo`注解表示当服务器有消息需要推送的时候，会对订阅了`@SendTo`中路径的客户端发送消息

# 前端脚本

我们这个案例需要三个js脚本文件，分别是STOMP协议的客户端脚本stomp.js、SockJS的客户端脚本sock.js以及jQuery

# 演示页面

```
<!DOCTYPE html>
<html>
<head>
<title>websocket</title>
<script src="//cdn.bootcss.com/angular.js/1.5.6/angular.min.js"></script>
<script src="https://cdn.bootcss.com/sockjs-client/1.1.4/sockjs.min.js"></script>
<script src="https://cdn.bootcss.com/stomp.js/2.3.3/stomp.min.js"></script>
<script type="text/javascript">
	/*<![CDATA[*/

	var stompClient = null;

	var app = angular.module('app', []);
	app.controller('MainController', function($rootScope, $scope, $http) {

		$scope.data = {
			//连接状态
			connected : false,
			//消息
			message : '',
			rows : []
		};

		//连接
		$scope.connect = function() {
			var socket = new SockJS('/my-websocket');
			stompClient = Stomp.over(socket);
			stompClient.connect({}, function(frame) {
				// 注册发送消息
				stompClient.subscribe('/topic/send', function(msg) {
					$scope.data.rows.push(JSON.parse(msg.body));
					$scope.data.connected = true;
					$scope.$apply();
				});
				// 注册推送时间回调
				stompClient.subscribe('/topic/callback', function(r) {
					$scope.data.time = '当前服务器时间：' + r.body;
					$scope.data.connected = true;
					$scope.$apply();
				});

				$scope.data.connected = true;
				$scope.$apply();
			});
		};

		$scope.disconnect = function() {
			if (stompClient != null) {
				stompClient.disconnect();
			}
			$scope.data.connected = false;
		}

		$scope.send = function() {
			stompClient.send("/app/send", {}, JSON.stringify({
				'message' : $scope.data.message
			}));
		}
	});
	/*]]>*/
</script>
</head>
<body ng-app="app" ng-controller="MainController">
	<label>WebSocket连接状态:</label>
	<button type="button" ng-disabled="data.connected" ng-click="connect()">连接</button>
	<button type="button" ng-click="disconnect()"
		ng-disabled="!data.connected">断开</button>
	<br />
	<br />
	<div ng-show="data.connected">
		<label>{{data.time}}</label> <br /> <br /> <input type="text"
			ng-model="data.message" placeholder="请输入内容..." />
		<button ng-click="send()" type="button">发送</button>
		<br /> <br /> 消息列表： <br />
		<table>
			<thead>
				<tr>
					<th>内容</th>
					<th>时间</th>
				</tr>
			</thead>
			<tbody>
				<tr ng-repeat="row in data.rows">
					<td>{{row.message}}</td>
					<td>{{row.date}}</td>
				</tr>
			</tbody>
		</table>
	</div>
</body>
</html>
```
![演示图](http://www.wailian.work/images/2018/03/02/WX20180302-094543.png)


[代码地址](https://github.com/wangweiye01/websocket)
