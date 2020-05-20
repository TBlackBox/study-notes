# 简介
可以通过tio的方式映入websocket


# 例子
1. 引入配置
```
<dependency>
    <groupId>org.t-io</groupId>
    <artifactId>tio-websocket-spring-boot-starter</artifactId>
    <version>3.5.5.v20191010-RELEASE</version>
</dependency>
```

2. 配置`handler`
```

import org.springframework.stereotype.Component;
import org.tio.core.ChannelContext;
import org.tio.http.common.HttpRequest;
import org.tio.http.common.HttpResponse;
import org.tio.websocket.common.WsRequest;
import org.tio.websocket.server.handler.IWsMsgHandler;

@Component
public class WebsocketHandler implements IWsMsgHandler {

	//握手
    @Override
    public HttpResponse handshake(HttpRequest httpRequest, HttpResponse httpResponse, ChannelContext channelContext) throws Exception {
        return httpResponse;
    }

    //握手成功
    @Override
    public void onAfterHandshaked(HttpRequest httpRequest, HttpResponse httpResponse, ChannelContext channelContext) throws Exception {
        System.out.println("握手成功");
    }

    //接收二进制文件
    @Override
    public Object onBytes(WsRequest wsRequest, byte[] bytes, ChannelContext channelContext) throws Exception {
        return null;
    }

    //断开连接
    @Override
    public Object onClose(WsRequest wsRequest, byte[] bytes, ChannelContext channelContext) throws Exception {
        System.out.println("关闭连接");
        return null;
    }

    //接收消息
    @Override
    public Object onText(WsRequest wsRequest, String s, ChannelContext channelContext) throws Exception {
    	System.out.println("接收文本消息:" + s);
        return "服务器接收到消息"+ s;
    }
}
```

3. 开启tiowebsocket服务
```
@SpringBootApplication
@EnableTioWebSocketServer  //开启tiowebsocket服务
public class StudyWebsocketTioApplication {

	public static void main(String[] args) {
		SpringApplication.run(StudyWebsocketTioApplication.class, args);
	}

}
```
`@EnableTioWebSocketServer`注解用于开始TioWebSocketServer服务使用，这里放到springboot启动类上面的，放到其他地方也是可以的。

# 总结
`TioWebSocket`是通过实现`IWsMsgHandler`接口进行事件处理的，里面不同的方法处理不同的时间，上面例子里面有简单的说明。这里只是列了服务端，客服端可以通过网站验证，[验证网站](http://coolaf.com/tool/chattest)