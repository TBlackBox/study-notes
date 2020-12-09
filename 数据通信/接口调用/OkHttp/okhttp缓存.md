# 简介
OkHttp实现了一个可选的，默认情况下关闭的Cache。OkHttp旨在实现RFC正确和实用的缓存行为，遵循Firefox / Chrome等常见的现实世界浏览器以及模棱两可时的服务器行为。

# 类
## Cache
将HTTP和HTTPS响应缓存到文件系统，以便它们可以重复使用，从而节省时间和带宽

# 缓存优化
缓存优化为了衡量缓存的有效性，此类跟踪三个统计信息：
- 请求计数：自创建此缓存以来发出的HTTP请求数。
- 网络计数：需要网络使用的那些请求的数量。
- 命中计数：高速缓存为其响应提供服务的那些请求的数量。
有时，请求将导致条件缓存命中。如果缓存包含响应的过期副本，则客户端将发出条件GET。然后，服务器将发送已更新的响应（如果已更改），或者发送简短的“未修改”响应（如果客户端的副本仍然有效）。这样的响应会同时增加网络计数和点击计数。提高缓存命中率的最佳方法是配置Web服务器以返回可缓存的响应。尽管此客户端使用所有HTTP / 1.1（RFC 7234）缓存头，但它不缓存部分响应。

# 强制网络响应
在某些情况下，例如在用户单击“刷新”按钮后，可能有必要跳过缓存并直接从服务器获取数据。要强制完全刷新，请添加no-cache指令：
```java
Request request = new Request.Builder()
       .cacheControl(new CacheControl.Builder().noCache().build())
       .url("http://publicobject.com/helloworld.txt")
       .build();
```
如果仅需要强制服务器验证缓存的响应，请改用效率更高的max-age = 0指令：
```java
Request request = new Request.Builder()
       .cacheControl(new CacheControl.Builder()
           .maxAge(0, TimeUnit.SECONDS)
           .build())
       .url("http://publicobject.com/helloworld.txt")
       .build();
```

# 强制执行缓存响应
有时，您可能想显示资源是否立即可用，但不是。可以使用它，以便您的应用程序可以在等待最新数据下载时显示某些内容。要将请求限制为本地缓存的资源，请添加only-if-cached指令：
```java
 Request request = new Request.Builder()
         .cacheControl(new CacheControl.Builder()
             .onlyIfCached()
             .build())
         .url("http://publicobject.com/helloworld.txt")
         .build();
     Response forceCacheResponse = client.newCall(request).execute();
     if (forceCacheResponse.code() != 504) {
       // The resource was cached! Show it.
     } else {
       // The resource was not cached.
     }
```
在过时的响应比没有响应要好的情况下，此技术效果更好。要允许过时的缓存响应，请使用max-stale指令，以秒为单位的最大过时度：
```java
 Request request = new Request.Builder()
       .cacheControl(new CacheControl.Builder()
           .maxStale(365, TimeUnit.DAYS)
           .build())
       .url("http://publicobject.com/helloworld.txt")
       .build();
```
`CacheControl`类可以配置请求缓存指令和解析响应缓存指令。它甚至提供了方便的常量CacheControl.FORCE_NETWORK和CacheControl.FORCE_CACHE，可以解决上述用例。

# CacheControl
具有来自服务器或客户端的缓存指令的Cache-Control标头。这些指令设置了关于可以存储哪些响应以及那些存储的响应可以满足哪些请求的策略。