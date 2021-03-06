# HTTP/1.1 的缺陷

## 1. 高延迟 – 带来页面加载速度的降低

虽然近几年来网络带宽增长非常快，然而我们却并没有看到网络延迟有对应程度的降低。**网络延迟问题主要由于队头阻塞 (Head-Of-Line Blocking) 产生，导致带宽无法被充分利用**。

### 队头阻塞
队头阻塞指当顺序发送的请求序列中的一个请求因为某种原因被阻塞时，在后面排队的所有请求也一并被阻塞，会导致客户端迟迟收不到数据。针对队头阻塞, 人们尝试过以下办法来解决：

- 将同一页面的资源分散到不同域名下，提升连接上限。**Chrome 有个机制，对于同一个域名，默认允许同时建立 6 个 TCP 持久连接。**使用持久连接时，虽然能共用一个 TCP 管道，**但是在一个管道中同一时刻只能处理一个请求**，在当前的请求没有结束之前，其他的请求只能处于阻塞状态。另外，如果在同一个域名下同时有 10 个请求发生，那么其中 4 个请求会进入排队等待状态，直至进行中的请求完成。
- Spriting 合并多张小图为一张大图, 再用 JavaScript 或者 CSS 将小图重新“切割”出来的技术。
- 内联 (Inlining) 是另一种防止发送很多小图请求的技巧，将图片的原始数据嵌入在 CSS 文件里面的 URL 里，减少网络请求次数。

```
.icon1 {
    background: url(data:image/png;base64,<data>) no-repeat;
  }
.icon2 {
    background: url(data:image/png;base64,<data>) no-repeat;
  }
```

- 拼接 (Concatenation) 将多个体积较小的 JavaScript 使用 webpack 等工具打包成 1 个体积更大的 JavaScript 文件, 但如果其中 1 个文件改动，会导致大量数据被重新下载多个文件。

## 2. 无状态特性 – 带来巨大的 HTTP 头部

由于报文 Header 一般会携带 "User Agent" “Cookie”“Accept”"Server" 等许多固定的头字段，多达几百字节甚至上千字节，但 Body 却经常只有几十字节（比如 GET 请求、 204/301/304 响应），成了不折不扣的“大头儿子”。Header 里携带的内容过大，在一定程度上增加了传输成本。更要命的是，成千上万的请求响应报文里有很多字段值都是重复的，非常浪费。

## 3. 明文传输 – 带来不安全性

HTTP/1.1 在传输数据时，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份，这在一定程度上无法保证数据的安全性。

你有没有听说过 " 免费 WiFi 陷阱”之类的新闻呢？黑客就是利用了 HTTP 明文传输的缺点，在公共场所架设一个 WiFi 热点开始“钓鱼”，诱骗网民上网。一旦你连上了这个 WiFi 热点，所有的流量都会被截获保存，里面如果有银行卡号、网站密码等敏感信息的话那就危险了，黑客拿到了这些数据就可以冒充你，为所欲为。

### 4. 不支持服务器推送消息
