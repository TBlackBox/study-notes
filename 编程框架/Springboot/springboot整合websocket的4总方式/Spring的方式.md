# 简介
通过spring的形式配置websocket,这里通过例子简单说明。


# 例子
* 引入依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

*  添加配置类
```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Autowired
    private HttpAuthHandler httpAuthHandler;
    
    @Autowired
    private WebsocketInterceptor websocketInterceptor;

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry
                .addHandler(httpAuthHandler, "ws")
                .addInterceptors(websocketInterceptor)
                .setAllowedOrigins("*");
    }
}
```
通过实现 **WebSocketConfigurer** 类并覆盖相应的方法进行 **websocket** 的配置。我们主要覆盖 **registerWebSocketHandlers** 这个方法。通过向 **WebSocketHandlerRegistry** 设置不同参数来进行配置。其中 **addHandler** 方法添加我们上面的写的 ws 的 handler 处理类，第二个参数是你暴露出的 ws 路径。**addInterceptors** 添加我们写的握手过滤器。**setAllowedOrigins("\*")** 这个是关闭跨域校验，方便本地调试，线上推荐打开。

* 添加事件处理
```

@Component
public class HttpAuthHandler extends TextWebSocketHandler {

    //socket 建立成功事件
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        Object token = session.getAttributes().get("token");
        if (token != null) {
            // 用户连接成功，放入在线用户缓存
            WsSessionManager.add(token.toString(), session);
        } else {
            throw new RuntimeException("用户登录已经失效!");
        }
    }

    //接收消息事件
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        // 获得客户端传来的消息
        String payload = message.getPayload();
        Object token = session.getAttributes().get("token");
        System.out.println("server 接收到 " + token + " 发送的 " + payload);
        session.sendMessage(new TextMessage("server 发送给 " + token + " 消息 " + payload + " " + LocalDateTime.now().toString()));
    }

    //socket 断开连接时
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        Object token = session.getAttributes().get("token");
        System.out.println("断开连接:"+ token);
        if (token != null) {
            // 用户退出，移除缓存
            WsSessionManager.remove(token.toString());
        }
    }
}
```

通过继承 **TextWebSocketHandler** 类并覆盖相应方法，可以对 websocket 的事件进行处理，这里可以同原生注解的那几个注解连起来看

1. **afterConnectionEstablished** 方法是在 socket 连接成功后被触发，同原生注解里的 @OnOpen 功能
2. **afterConnectionClosed** 方法是在 socket 连接关闭后被触发，同原生注解里的 @OnClose 功能
3. **handleTextMessage** 方法是在客户端发送信息时触发，同原生注解里的 @OnMessage 功能

* 配置拦截器
```
@Component
public class WebsocketInterceptor implements HandshakeInterceptor {

    //握手前 
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
        System.out.println("握手开始");
        // 获得请求参数
        HashMap<String, String> paramMap = HttpUtil.decodeParamMap(request.getURI().getQuery(), "utf-8");
        String uid = paramMap.get("token");
        //TODO:方便测试  这点固定
        uid = "1";
        if (StrUtil.isNotBlank(uid)) {
            // 放入属性域
            attributes.put("token", uid);
            System.out.println("用户 token " + uid + " 握手成功！");
            return true;
        }
        System.out.println("用户登录已失效");
        return false;
    }

    //握手后
    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {
        System.out.println("握手完成");
    }
}
	
```

通过实现 **HandshakeInterceptor** 接口来定义握手拦截器，注意这里与上面 **Handler** 的事件是不同的，这里是建立握手时的事件，分为握手前与握手后，而 **Handler** 的事件是在握手成功后的基础上建立 socket 的连接。所以在如果把认证放在这个步骤相对来说最节省服务器资源。它主要有两个方法 **beforeHandshake** 与 **afterHandshake** ，顾名思义一个在握手前触发，一个在握手后触发

* 添加session缓存类

  这里简单通过 **ConcurrentHashMap** 来实现了一个 session 池，用来保存已经登录的 web socket 的 session。服务端发送消息给客户端必须要通过这个 session。
```
//session 管理
public class WsSessionManager {
    
	//保存连接 session 的地方
    private static ConcurrentHashMap<String, WebSocketSession> SESSION_POOL = new ConcurrentHashMap<>();

    //添加 session
    public static void add(String key, WebSocketSession session) {
        // 添加 session
        SESSION_POOL.put(key, session);
    }

    //删除 session,会返回删除的 session
    public static WebSocketSession remove(String key) {
        // 删除 session
        return SESSION_POOL.remove(key);
    }

    //删除并同步关闭连接
    public static void removeAndClose(String key) {
        WebSocketSession session = remove(key);
        if (session != null) {
            try {
                // 关闭连接
                session.close();
            } catch (IOException e) {
                // todo: 关闭出现异常处理
                e.printStackTrace();
            }
        }
    }

    //获得 session
    public static WebSocketSession get(String key) {
        // 获得 session
        return SESSION_POOL.get(key);
    }
}
```

* 启动
```
@SpringBootApplication
public class StudyWebsocketSpringApplication {
	public static void main(String[] args) {
		SpringApplication.run(StudyWebsocketSpringApplication.class, args);
	}
}
```
访问地址
```
ws://127.0.0.1:8080/ws
```

# 总结
推荐使用这种配置进行springboot websocket配置，使得应用更加的灵活，思路更加的清晰。
[在线测试地址](http://coolaf.com/tool/chattest)
