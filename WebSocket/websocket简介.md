# 简介
WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

现在，很多网站为了实现推送技术，所用的技术都是 Ajax 轮询。轮询是在特定的的时间间隔（如每1秒），由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这种传统的模式带来很明显的缺点，即浏览器需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。

HTML5 定义的 WebSocket 协议，能更好的节省服务器资源和带宽，并且能够更实时地进行通讯。

浏览器通过 JavaScript 向服务器发出建立 WebSocket 连接的请求，连接建立以后，客户端和服务器端就可以通过 TCP 连接直接交换数据。

当你获取 Web Socket 连接后，你可以通过 send() 方法来向服务器发送数据，并通过 onmessage 事件来接收服务器返回的数据。

[菜鸟教程](https://www.runoob.com/html/html5-websocket.html)
[developer](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)


# WebSocket 
WebSocket 对象提供了用于创建和管理 WebSocket 连接，以及可以通过该连接发送和接收数据的 API。

## 创建WebSocket对象
```
var Socket = new WebSocket(url, [protocol] );
```
以上代码中的第一个参数 url, 指定连接的 URL。第二个参数 protocol 是可选的，指定了可接受的子协议。



## WebSocket 属性

以下是 WebSocket 对象的属性。假定我们使用了以上代码创建了 Socket 对象：

## 属性

以下是 WebSocket 对象的属性。假定我们使用了以上代码创建了 Socket 对象：

- `WebSocket.binaryType`

  使用二进制的数据类型连接。

- `WebSocket.bufferedAmount` 只读

  未发送至服务器的字节数。只读属性 **bufferedAmount** 已被 send() 放入正在队列中等待传输，但是还没有发出的 UTF-8 文本字节数。

- `WebSocket.extensions` 只读

  服务器选择的扩展。

- `WebSocket.onclose`

  用于指定连接关闭后的回调函数。

- `WebSocket.onerror`

  用于指定连接失败后的回调函数。

- `WebSocket.onmessage`

  用于指定当从服务器接受到信息时的回调函数。

- `WebSocket.onopen`

  用于指定连接成功后的回调函数。

- `WebSocket.protocol` 只读

  服务器选择的下属协议。

- [`WebSocket.readyState`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/readyState) 只读

  当前的链接状态。

  只读属性 **readyState** 表示连接状态，可以是以下值：
    - 0 表示连接尚未建立
    - 1 表示连接已建立，可以进行通信。
    - 2 表示连接正在进行关闭。
    - 3 表示连接已经关闭或者连接不能打开。

- `WebSocket.url` 只读

  WebSocket 的绝对路径。

## 常量

| **Constant**           | **Value** |
| ---------------------- | --------- |
| `WebSocket.CONNECTING` | `0`       |
| `WebSocket.OPEN`       | `1`       |
| `WebSocket.CLOSING`    | `2`       |
| `WebSocket.CLOSED`     | `3`       |


## 方法

| 方法           | 描述             |
| :------------- | :--------------- |
| Socket.send()  | 使用连接发送数据 对要传输的数据进行排队。 |
| Socket.close() | 关闭当前链接。       |


## 事件
以下是 WebSocket 对象的相关事件。假定我们使用了以上代码创建了 Socket 对象：

| 事件    | 事件处理程序     | 描述                       |
| :------ | :--------------- | :------------------------- |
| open    | Socket.onopen    | 连接建立时触发             |
| message | Socket.onmessage | 客户端接收服务端数据时触发 |
| error   | Socket.onerror   | 通信发生错误时触发         |
| close   | Socket.onclose   | 连接关闭时触发             |

# 案列
WebSocket 协议本质上是一个基于 TCP 的协议。

为了建立一个 WebSocket 连接，客户端浏览器首先要向服务器发起一个 HTTP 请求，这个请求和通常的 HTTP 请求不同，包含了一些附加头信息，其中附加头信息"Upgrade: WebSocket"表明这是一个申请协议升级的 HTTP 请求，服务器端解析这些附加的头信息然后产生应答信息返回给客户端，客户端和服务器端的 WebSocket 连接就建立起来了，双方就可以通过这个连接通道自由的传递信息，并且这个连接会持续存在直到客户端或者服务器端的某一方主动的关闭连接。

```
var SocketBox = {
		socket:null,
		isAuth:false,
		
		init:function(fromId,url){
			if(!window.WebSocket){
				window.WebSocket = window.MozWebSocket;
			}
			if(window.WebSocket){
				socket = new WebSocket(url);
				socket.onmessage = function(event){
					var data = eval("(" + event.data + ")");
					// console.log("onmessage data: " + JSON.stringify(data));
					switch (data.code) {
						case 1 << 8 | 220: // ping message
						case 2 << 8 | 220: // pong message
							console.log("ping message: " + JSON.stringify(data));
							SocketBox.pingInvake(data);
							break;
						case 3 << 8 | 220: // system message
							console.log("system message: " + JSON.stringify(data));
							SocketBox.sysInvake(data);
							break;
						case 4 << 8 | 220: // error message  错误消息
							console.log("error message: " + JSON.stringify(data));
							SocketBox.closeInvake(null);
							break;
						case 5 << 8 | 220: // auth message   //认证消息 
							console.log("auth message: " + JSON.stringify(data));
							break;
						case 6 << 8 | 220: // 普通消息  也是业务逻辑消息
							console.log("broadcast message: " + JSON.stringify(data));
							SocketBox.messageInvake(data);
							break;

					}
				};
				socket.onopen = function(event){
					console.log("Netty-WebSocket服务器。。。。。。连接  \r\n");
					//服务器连接成功，进行认证
					SocketBox.openInvake(fromId,event);
				};

				//socket 服务器关闭
				socket.onclose = function(event){
					console.log("Netty-WebSocket服务器。。。。。。关闭");
					SocketBox.closeInvake(event);
				};

				//连接出错
				socket.onerror = function (ev) {
					console.log(ev.data + "WebSocket连接异常\r\n");
				};
			}else{
				alert("您的浏览器不支持WebSocket协议！");
			}
		},
		pingInvake:function(data){
			//处理ping消息
			SocketBox.sendData(this.isAuth, "{'code':10016}");
		},
		sysInvake:function(data){
			//处理系统消息
			switch (data.extend.code) {
				case 20001: // user count
					console.log("current user: " + data.extend.mess);
					userCount = data.extend.mess;
					console.log("当前在线人数:" + userCount);
					break;
				case 20002: // auth
					console.log("auth result: " + data.extend.mess);
					isAuth = data.extend.mess;
					break;
				case 20003: // system message
					console.log("system message: " + data.extend.mess);
					break;
			}

		},
		messageInvake:function(data){
			//处理普通消息
			var body = data.body;
			console.log("接受到普通消息：",data);
			if(body.type == MessageCode.PEER_MESSAGE){
				//点对点私人消息
				answerTextMessage(body);
			}
			/*if(data.extend.code == 30001 && data.extend.data.messageType == 1000){
				answerTextMessage(mess);
			}*/
		},
		sendData(auth, mess){
			//发送数据
			if (!window.socket) {
				return;
			}
			if (socket.readyState == WebSocket.OPEN || auth) {
				console.log("最终发送的数据: " + mess);
				window.socket.send(mess);
			} else {
				console.log("连接没有成功，请重新登录");
			}
		},
		//发送点对点文本消息
		sendPeerTextMessage:function(data){
			//发送点对点文本消息
            SocketBox.sendData(true, "{'code':"+MessageCode.MESS_CODE+",'data':'"+JSON.stringify(data)+"'}");
		},
		openInvake:function(fromId,event){
			//通道打开
			var obj = {};
			obj.code = 10000;
			obj.id = fromId;
			SocketBox.sendData(true, JSON.stringify(obj));
		},
		closeInvake:function(event){
			this.socket = null;
			this.isAuth = false;
			console.log("登录失败，网络连接异常,event:",event);
		}
		
};

//在刷新页面或者关闭的页面的时候关闭socket
window.onbeforeunload = function() {
	//关闭socket
	SocketBox.socket.close();
}
```
