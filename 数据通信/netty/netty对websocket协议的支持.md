

## 初始化链接的组件
```java
public class ChannelHandler extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        System.out.println("收到新连接");
        //websocket协议本身是基于http协议的，所以这边也要使用http解编码器
        ch.pipeline().addLast(new HttpServerCodec());
        //以块的方式来写的处理器
        ch.pipeline().addLast(new ChunkedWriteHandler());
        //netty是基于分段请求的，HttpObjectAggregator的作用是将请求分段再聚合,参数是聚合字节的最大长度
        ch.pipeline().addLast(new HttpObjectAggregator(8192));
        //ws://server:port/context_path
        //ws://localhost:9999/ws
        //参数指的是contex_path
        ch.pipeline().addLast(new WebSocketServerProtocolHandler("/ws", "WebSocket", true, 65536 * 10));

        //实现心跳
        ch.pipeline().addLast(new IdleStateHandler(600, 0, 0, TimeUnit.SECONDS));
//        ch.pipeline().addLast(new HeartBeatHandler());
//
//        //自定义助手类
//        ch.pipeline().addLast(new WebSocketHandler());
//
//        //消息处理器类
//        ch.pipeline().addLast(new MessageHandler());
    }
}
```

## `HttpServerCodec`
将字节解码为HttpRequest，HttpContent和LastHttpContent，并将HttpRequest，HttpContent和LastHttpContent编码为字节。

## `ChunkedWriteHandler`
一个支持异步编写大型数据流的支持的handler，既不消耗大量内存也不会产生OutOfMemoryError。 诸如文件传输的大数据流需要在ChannelHandler实现中进行复杂的状态管理。 ChunkedWriteHandler管理这样复杂的状态，以便您可以毫无困难地发送大数据流。换句话说，就是netty提供的支持大文件数据写的一个处理器。


## `WebSocketServerProtocolHandler`

`WebSocketServerProtocolHandler`：参数是访问路径，这边指定的是ws，服务客户端访问服务器的时候指定的url是：`ws://localhost:8899/ws`。
 它负责websocket握手以及处理控制框架（Close，Ping（心跳检检测request），Pong（心跳检测响应））。 文本和二进制数据帧被传递到管道中的下一个处理程序进行处理。



## **桢**：
 WebSocket规范中定义了6种类型的桢，由IETF 发布的WebSocket RFC，定义了6 种帧，Netty 为它们每种都提供了一个POJO 实现，下面我们来看一下 WebSocketFrame 六种不同的类型，前三种为数据帧，后三种为控制帧，其中最后两种帧为心跳报文，ping发起心跳，pong回应心跳。
 ## WebSocketFrame
 所有桢的父类，所谓桢就是WebSocket服务在建立的时候，在通道中处理的数据类型。

 ## TextWebSocketFrame
 包含了文本数据，处理文本协议数据，处理TextWebSocketFrame类型的数据，websocket专门处理文本的frame就是TextWebSocketFrame。


## BinaryWebSocketFrame
包含了二进制数据

## ContinuationWebSocketFrame

包含属于上一个BinaryWebSocketFrame或者TextWebSocketFrame的文本数据或者二进制数据

## CloseWebSocketFrame
表示一个CLOSE请求，包含一个关闭的状态码和关闭的原因

## PingWebSocketFrame

请求传输一个PongWebSocketFrame



## PongWebSocketFrame
作为一个对于PingWebSocketFrame的响应被发送

