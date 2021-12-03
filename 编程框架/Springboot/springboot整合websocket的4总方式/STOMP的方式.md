# 简介 
通过stomp协议的方式配置websocket


# 例子
1. 引入依赖
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

2. 创建配置类
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 配置客户端尝试连接地址
        registry.addEndpoint("/ws").setAllowedOrigins("*").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // 设置广播节点
        registry.enableSimpleBroker("/topic", "/user");
        
        // 客户端向服务端发送消息需有/app 前缀
        registry.setApplicationDestinationPrefixes("/app");
        // 指定用户发送（一对一）的前缀 /user/
        registry.setUserDestinationPrefix("/user/");
    }
}
```

3. 创建controller
```java
@Controller
public class WSController {

    @Autowired
    private SimpMessagingTemplate simpMessagingTemplate;

    @MessageMapping("/hello")
    @SendTo("/topic/hello")
    public ResponseMsg hello(String requestMessage) {
        System.out.println("接收消息：" + requestMessage);
        return new ResponseMsg("服务端接收到你发的：" + requestMessage);
    }

    @GetMapping("/sendMsgByUser")
    public @ResponseBody
    Object sendMsgByUser(String token, String msg) {
        simpMessagingTemplate.convertAndSendToUser(token, "/msg", msg);
        return "success";
    }

    @GetMapping("/sendMsgByAll")
    public @ResponseBody
    Object sendMsgByAll(String msg) {
        simpMessagingTemplate.convertAndSend("/topic", msg);
        return "success";
    }

    @GetMapping("/test")
    public String test() {
        return "test-stomp.html";
    }
}

```

- 通过 **@MessageMapping** 来暴露节点路径，有点类似 **@RequestMapping**。注意这里虽然写的是 hello ，但是我们客户端调用的真正地址是 **/app/hello**。 因为我们在上面的 config 里配置了**registry.setApplicationDestinationPrefixes("/app")**。

- **@SendTo**这个注解会把返回值的内容发送给订阅了 **/topic/hello** 的客户端，与之类似的还有一个**@SendToUser** 只不过他是发送给用户端一对一通信的。这两个注解一般是应答时响应的，如果服务端主动发送消息可以通过 **simpMessagingTemplate**类的**convertAndSend**方法。注意 **simpMessagingTemplate.convertAndSendToUser(token, "/msg", msg)** ，联系到我们上文配置的 **registry.setUserDestinationPrefix("/user/"),**这里客户端订阅的是**/user/{token}/msg**,千万不要搞错。

4. 简单的消息实体
```java
public class ResponseMsg {

	private String responseBody;

	public String getResponseBody() {
		return responseBody;
	}
	
	public ResponseMsg(String responseBody) {
		this.responseBody = responseBody;
	}

	public void setResponseBody(String responseBody) {
		this.responseBody = responseBody;
	}
}
```

5. 简单的html测试页面
```java
<html>
<head>
    <meta charset="UTF-8"/>
    <title>Stomp 测试</title>
    <script src="https://lib.baomitu.com/sockjs-client/1.4.0/sockjs.min.js"></script>
    <script src="https://lib.baomitu.com/stomp.js/2.3.3/stomp.min.js"></script>
    <script src="https://lib.baomitu.com/jquery/3.4.1/jquery.min.js"></script>
</head>
<body onload="disconnect()">
<noscript><h2 style="color: #e80b0a;">Sorry，浏览器不支持WebSocket</h2></noscript>
<div>
    <div>
        <button id="connect" onclick="connect();">连接</button>
        <button disabled="disabled" id="disconnect" onclick="disconnect();">断开连接</button>
    </div>
    <div id="conversationDiv">
        <label>发送信息</label><input id="name" type="text"/>
        <button id="sendName" onclick="sendName();">发送</button>
        <p id="response"></p>
        <p id="callback"></p>
    </div>
</div>
<script type="text/javascript">
    var stompClient = null;

    function setConnected(connected) {
        document.getElementById("connect").disabled = connected;
        document.getElementById("disconnect").disabled = !connected;
        document.getElementById("conversationDiv").style.visibility = connected ? 'visible' : 'hidden';
        $("#response").html();
        $("#callback").html();
    }

    function connect() {
        var socket = new SockJS('http://localhost:8080/ws');
        stompClient = Stomp.over(socket);
        stompClient.connect({}, function (frame) {
            setConnected(true);
            console.log('Connected:' + frame);
            stompClient.subscribe('/topic/hello', function (response) {
                showResponse(JSON.parse(response.body).responseMessage);
            });
            // 另外再注册一下定时任务接受
            stompClient.subscribe('/user/123/msg', function (response) {
                showCallback(response.body);
            });
        });
    }

    function disconnect() {
        if (stompClient != null) {
            stompClient.disconnect();
        }
        setConnected(false);
        console.log('Disconnected');
    }

    function sendName() {
        var name = $('#name').val();
        console.log('name:' + name);
        stompClient.send("/app/hello", {}, JSON.stringify({'name': name}));
    }

    function showResponse(message) {
        $("#response").html(message);
    }

    function showCallback(message) {
        $("#callback").html(message);
    }
</script>
</body>
</html>
```

# 配置说明

通过实现 **WebSocketMessageBrokerConfigurer** 接口和加上**@EnableWebSocketMessageBroker**来进行 stomp 的配置与注解扫描。

其中覆盖 **registerStompEndpoints** 方法来设置暴露的 stomp 的路径，其它一些跨域、客户端之类的设置。

覆盖 **configureMessageBroker** 方法来进行节点的配置。

1. 其中 **enableSimpleBroker** 配置的广播节点，也就是服务端发送消息，客户端订阅就能接收消息的节点。
2. 覆盖**setApplicationDestinationPrefixes** 方法，设置客户端向服务端发送消息的节点。
3. 覆盖 **setUserDestinationPrefix** 方法，设置一对一通信的节点。

