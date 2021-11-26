# 简介
本文档主要用于初学者创建netty案列学习和测试使用。本案例用到的maven版本是4.4.38，上面的很多注释解释了netty的使用方法，线程模型登，是netty入门的不二之选。

# 添加项目依赖
新建maven工程，并在破灭。xml里面添加项目依赖。
```xml
<properties>
    <netty-all.version>4.1.38.Final</netty-all.version>
</properties>

<dependencies>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>${netty-all.version}</version>
    </dependency>
</dependencies>
```

# 常见服务端的环境
这里都都不详解的介绍包的结构了，安需安排就可以了。

## 常见SocketServer,用于初始化socket配置和启动
```java
/**
 * 1. 双线程组
 * 2. Bootstrap配置启动信息
 * 3. 注册业务处理Handler
 * 4. 绑定服务监听端口并启动服务
 */
package cn.javafroum.netty.server;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

/**
 * @author BlackBox
 */
public class SocketServer {
	// 监听线程组，监听客户端请求
	private EventLoopGroup acceptorGroup = null;
	// 处理客户端相关操作线程组，负责处理与客户端的数据通讯
	private EventLoopGroup clientGroup = null;
	// 服务启动相关配置信息
	private ServerBootstrap bootstrap = null;
	public SocketServer(){
		init();
	}
	private void init(){
		//这里将一下线程模型的配置
		//netty里面有3种线程模型 1.单线程模型 2.多线程模型  3.主从多线程模型
		//1. 单线程模型，即是接受线程组和处理线程组都为同一个并且只有一个线程，只需要更改如下配置
		//acceptorGroup = new NioEventLoopGroup();
		//bootstrap.group(acceptorGroup, acceptorGroup);

		//2.多线程模型，即是接受线程只有一个，处理线程有多个
		//acceptorGroup = new NioEventLoopGroup(1);
		//clientGroup = new NioEventLoopGroup();
		//bootstrap.group(acceptorGroup, clientGroup);

		//3.主从多线程模型,即是下面这种配置，接受线程多个，处理线程也是多个
		// 初始化线程组,构建线程组的时候，如果不传递参数，则默认构建的线程组线程数是CPU核心数量。
		acceptorGroup = new NioEventLoopGroup();
		clientGroup = new NioEventLoopGroup();
		// 初始化服务的配置
		bootstrap = new ServerBootstrap();
		// 绑定线程组
		bootstrap.group(acceptorGroup, clientGroup);
		// 设定通讯模式为NIO， 同步非阻塞
		bootstrap.channel(NioServerSocketChannel.class);
		// 设定缓冲区大小， 缓存区的单位是字节。
		bootstrap.option(ChannelOption.SO_BACKLOG, 1024);
		// SO_SNDBUF发送缓冲区
		// SO_RCVBUF接收缓冲区
		// SO_KEEPALIVE开启心跳监测（保证连接有效）
		bootstrap.option(ChannelOption.SO_SNDBUF, 16*1024)
			.option(ChannelOption.SO_RCVBUF, 16*1024)
			.option(ChannelOption.SO_KEEPALIVE, true);
	}

	/**
	 * 监听处理逻辑。
	 * @param port 监听端口。
	 * @param acceptorHandlers 处理器， 如何处理客户端请求。
	 * @return
	 * @throws InterruptedException
	 */
	public ChannelFuture doAccept(int port, final ChannelHandler... acceptorHandlers) throws InterruptedException{
		
		/*
		 * childHandler是服务的Bootstrap独有的方法。是用于提供处理对象的。
		 * 可以一次性增加若干个处理逻辑。是类似责任链模式的处理方式。
		 * 增加A，B两个处理逻辑，在处理客户端请求数据的时候，根据A-》B顺序依次处理。
		 * 
		 * ChannelInitializer - 用于提供处理器的一个模型对象。
		 *  其中定义了一个方法，initChannel方法。
		 *   方法是用于初始化处理逻辑责任链条的。
		 *   可以保证服务端的Bootstrap只初始化一次处理器，尽量提供处理逻辑的重用。
		 *   避免反复的创建处理器对象。节约资源开销。
		 */
		bootstrap.childHandler(new ChannelInitializer<SocketChannel>() {

			@Override
			protected void initChannel(SocketChannel ch) throws Exception {
				ch.pipeline().addLast(acceptorHandlers);
			}
		});
		// bind方法 - 绑定监听端口的。ServerBootstrap可以绑定多个监听端口。 多次调用bind方法即可
		// sync - 开始监听逻辑。 返回一个ChannelFuture。 返回结果代表的是监听成功后的一个对应的未来结果
		// 可以使用ChannelFuture实现后续的服务器和客户端的交互。
		ChannelFuture future = bootstrap.bind(port).sync();
		return future;
	}
	
	/**
	 * shutdownGracefully - 方法是一个安全关闭的方法。可以保证不放弃任何一个已接收的客户端请求。
	 */
	public void release(){
		this.acceptorGroup.shutdownGracefully();
		this.clientGroup.shutdownGracefully();
	}
}

```

## 参见测试handler,方便我们处理客服端传来的消息
```java
package cn.javafroum.netty.server.handler;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.io.UnsupportedEncodingException;


//Sharable 注解的说明：
//作用:多个客服端可以共用这个实例，但要注意的是，不能再里面声明变量，这样不安全，声明了变量的话，
//多个客服端用这边变量会导致不安全
@ChannelHandler.Sharable
public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)

    /**
     * 业务处理逻辑
     * 用于处理读取数据请求的逻辑。
     * msg - 读取到的数据。 默认类型是ByteBuf，是Netty自定义的。是对ByteBuffer的封装。 不需要考虑复位问题。
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws UnsupportedEncodingException { // (2)

        //ctx - 上下文对象。其中包含于客户端建立连接的所有资源。 如： 对应的Channel
        System.out.println("通道id:" + ctx.channel().id().asLongText());


        // 获取读取的数据， 是一个缓冲。
        ByteBuf readBuffer = (ByteBuf) msg;
        // 创建一个字节数组，用于保存缓存中的数据。
        byte[] tempDatas = new byte[readBuffer.readableBytes()];
        // 将缓存中的数据读取到字节数组中。
        readBuffer.readBytes(tempDatas);
        String message = new String(tempDatas, "UTF-8");
        System.out.println("收到客服端的消息 : " + message);
        if("exit".equals(message)){
            ctx.close();
            return;
        }
        String line = "服务端接收到了客服端的消息 : " + message + "  这是返回确认";
        // 写操作自动释放缓存，避免内存溢出问题。
        ctx.writeAndFlush(Unpooled.copiedBuffer(line.getBytes("UTF-8")));
        // 注意，如果调用的是write方法。不会刷新缓存，缓存中的数据不会发送到客户端，必须再次调用flush方法才行。
        // ctx.write(Unpooled.copiedBuffer(line.getBytes("UTF-8")));
        // ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        System.out.println("server exceptionCaught method run...");
        cause.printStackTrace();
        ctx.close();
    }
}
```

## 创建启动类
```java
package cn.javafroum.netty.server;

import cn.javafroum.netty.server.handler.DiscardServerHandler;
import io.netty.channel.ChannelFuture;

public class StartupServer {
    public static void main(String[] args){
        ChannelFuture future = null;
        SocketServer server = null;
        try{
            server = new SocketServer();
            future = server.doAccept(9090,new DiscardServerHandler());
            System.out.println("socket 服务端开始运行！");
            // 关闭连接的。
            future.channel().closeFuture().sync();
        }catch(InterruptedException e){
            e.printStackTrace();
        }finally{
            if(null != future){
                try {
                    future.channel().closeFuture().sync();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            if(null != server){
                server.release();
            }
        }
    }
}

```


# 客服端结构创建

## 创建客服端配置和初始化类
```java
/**
 * 1. 单线程组
 * 2. Bootstrap配置启动信息
 * 3. 注册业务处理Handler
 * 4. connect连接服务，并发起请求
 */
package cn.javafroum.netty.client;

import java.util.Scanner;
import java.util.concurrent.TimeUnit;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

/**
 * 因为客户端是请求的发起者，不需要监听。
 * 只需要定义唯一的一个线程组即可。
 */
public class SocketClient {
	
	// 处理请求和处理服务端响应的线程组
	private EventLoopGroup group = null;
	// 客户端启动相关配置信息
	private Bootstrap bootstrap = null;
	
	public SocketClient(){
		init();
	}
	
	private void init(){
		group = new NioEventLoopGroup();
		bootstrap = new Bootstrap();
		// 绑定线程组
		bootstrap.group(group);
		// 设定通讯模式为NIO
		bootstrap.channel(NioSocketChannel.class);
	}
	
	public ChannelFuture doRequest(String host, int port, final ChannelHandler... handlers) throws InterruptedException{
		/*
		 * 客户端的Bootstrap没有childHandler方法。只有handler方法。
		 * 方法含义等同ServerBootstrap中的childHandler
		 * 在客户端必须绑定处理器，也就是必须调用handler方法。
		 * 服务器必须绑定处理器，必须调用childHandler方法。
		 */
		this.bootstrap.handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel ch) throws Exception {
				ch.pipeline().addLast(handlers);
			}
		});
		// 建立连接。
		ChannelFuture future = this.bootstrap.connect(host, port).sync();
		return future;
	}
	
	public void release(){
		this.group.shutdownGracefully();
	}
}

```

## 创建客服端的handler，用于处理接收到服务端的返回消息
```java
package cn.javafroum.netty.client.handler;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerAdapter;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.ReferenceCountUtil;

public class ClientHandler extends ChannelInboundHandlerAdapter {

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		try{
			ByteBuf readBuffer = (ByteBuf) msg;
			byte[] tempDatas = new byte[readBuffer.readableBytes()];
			readBuffer.readBytes(tempDatas);
			System.out.println("from server : " + new String(tempDatas, "UTF-8"));
		}finally{
			// 用于释放缓存。避免内存溢出
			ReferenceCountUtil.release(msg);
		}
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		System.out.println("client exceptionCaught method run...");
		// cause.printStackTrace();
		ctx.close();
	}

	/*@Override // 断开连接时执行
	public void channelInactive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("channelInactive method run...");
	}

	@Override // 连接通道建立成功时执行
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("channelActive method run...");
	}

	@Override // 每次读取完成时执行
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		System.out.println("channelReadComplete method run...");
	}*/

}

```

## 创建客服端的启动类
```java
package cn.javafroum.netty.client;

import cn.javafroum.netty.client.handler.ClientHandler;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;

import java.util.Scanner;
import java.util.concurrent.TimeUnit;

public class StartupClient {
    public static void main(String[] args) {
        SocketClient client = null;
        ChannelFuture future = null;
        try{
            client = new SocketClient();
            future = client.doRequest("localhost", 9090, new ClientHandler());

            Scanner s = null;
            while(true){
                s = new Scanner(System.in);
                System.out.print("enter message send to server (enter 'exit' for close client) > ");
                String line = s.nextLine();
                if("exit".equals(line)){
                    // addListener - 增加监听，当某条件满足的时候，触发监听器。
                    // ChannelFutureListener.CLOSE - 关闭监听器，代表ChannelFuture执行返回后，关闭连接。
                    future.channel().writeAndFlush(Unpooled.copiedBuffer(line.getBytes("UTF-8")))
                            .addListener(ChannelFutureListener.CLOSE);
                    break;
                }
                future.channel().writeAndFlush(Unpooled.copiedBuffer(line.getBytes("UTF-8")));
                TimeUnit.SECONDS.sleep(1);
            }
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            if(null != future){
                try {
                    future.channel().closeFuture().sync();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            if(null != client){
                client.release();
            }
        }
    }
}

```


# 总结
按照上面的配置创建好相应的客服端和服务端，只需要运行启动类就可以了，代码里面有比较详细的注释，文档上面也都不说明了，源码可以参照   
[github学习案列](https://github.com/TBlackBox/study-example) 

