# ** 1. 简介**
```
abstract public class HttpURLConnection extends URLConnection {}
```
在java.net包下的一个抽象类。

# **2. HttpURLConnection**

java.net.HttpURLConnection类是URLConnection的抽象子类；它提供了一些有助于专门操作http URL的额外方法。

sun.net.www.protocol.http包中也提供了一个HttpURLConnection类。该类实现了抽象的connect()方法。但一般很少直接访问该类。



## **2.1 获取HttpURLConnection对象**

```
@Test
void testHttpURLConnection(){
    try {
        URL url = new URL("http:www.baidu.com");
        URLConnection urlConnection = url.openConnection();
        HttpURLConnection httpURLConnection = (HttpURLConnection)urlConnection;
    } catch (MalformedURLException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

 

 

## **2.2 请求方法**

HTTP协议提供了7中请求方法：

- GET
- POST
- HEAD
- PUT
- OPTIONS
- DELETE
- TRACE

通过以下方法设置请求方法：

```
void setRequestMethod(String method)

````


若请求不是7个方法的字符串，则会抛出java.net.ProtocolException异常。一般只设置请求方法是不够的，还需要调整HTTP首部，还要提供消息体。

以下方法获取HTTP请求方法

```
String getRequestMethod()
```




## **2.3 断开与服务器的连接**

HTTP的Keep-Alive对Web连接的性能有所提升，允许多个请求和响应通过一个TCP连接连续发送。

客户端通过HTTP请求首部包含Connection字段及Keep-Alive，表示客户端愿意使用Keep-Alive：

```
Connection: Keep-Alive
```


但当使用Keep-Alive时，服务器不再因为向客户端发送最后一个字节数据就关闭连接。毕竟客户端还可能发送另一个请求。因此客户端需要复杂关闭连接。

Java提供如下方法关闭连接，实际上很少使用该方法：

```
abstract void disconnect()
```


## **2.4 获取请求**

如果HTTP响应状态为4xx（客户端错误）或者5xx（服务器错误），你可以通过HttpUrlConnection.getErrorStream()来查看服务器发送过来的信息。

```
InputStream getErrorStream()
```


例：

```
InputStream error = ((HttpURLConnection) connection).getErrorStream();
```

 

如果HTTP响应状态为-1，就是出现了连接错误。HttpURLConnection会保持连接一直可用，如果你想关闭这个特性，需要把http.keepAlive设置为false：

```
System.setProperty("http.keepAlive", "false");
```

 

 

## **2.5 Http响应码**

HTTP响应的第一行被称为消息响应。该信息不会被URLConnection的getHeaderFiled()方法读取。

```
String getResponseMessage()
```

　　返回消息响应。如：HTTP/1.1 404 Not Found

```
int getResponseCode()
```

　　返回消息响应码。如：404。

HttpURLConnection类提供了常见响应码常量。具体请参考JDK文档。

 

 

## **2.6 重定向**

默认情况下，HttpURLConnection类会跟踪重定向。但HttpURLConnection有两个静态方法，可以确定是否跟随重定向。

```
static void setFollowRedirects(boolean set)

static boolean getFollowRedirects()
```

　　若跟随，则为true，否则为false。

 

这两个是静态方法，它们会改变该方法后构造的所有HttpURLConnection对象的行为。若安全管理器不允许其改变，则会抛出SecurityException异常。

 

也可以对某个HttpURLConnection实例设置是否重定向。

```
void setInstanceFollowRedirects(boolean followRedirects)

boolean getInstanceFollowRedirects()
```




## **2.7 代理**

HttpURLConnection可以得知请求是否通过了代理服务器。

使用如下方法：

```
abstract boolean usingProxy()
```


## **2.8 流模式**

每个发送给HTTP服务器的请求都有HTTP首部。首部中有一个Content-type字段；即请求体中的字节数。首部位于主体的前面。但是，为了写入首部，需要知道主体的长度，而在写首部的时候可能还不知道主题的长度。一般情况下Java对此两难境地的解决办法是，对于从HttpURLConnection获取的OutputStream，缓存写入到此OutputStream中的所有内容，直到流被关闭。到这时，它就知道了主题中有多少字节，

在请求发送之前，HttpURLConnetion会把所有需要发送的数据放到缓冲区里，不管你是否使用connection.setRequestProperty("Content-Length", contentLength);设置了contentLength，当你并行发送大量的POST请求时，这可能会引起OutOfMemoryExceptions 异常，为了避免这个问题，需要使用如下方法：

```
void setFixedLengthStreamingMode(int contentLength)
```

　　此方法用于在预先已知内容长度时启用没有进行内部缓冲的 HTTP 请求正文的流。

例：

```
httpConnection.setFixedLengthStreamingMode(contentLength);
```

 

但是如果不能事先知道内容的长度，可以使用HttpURLConnection.setChunkedStreamingMode()方法设置为块状流模式。在块状流模式的情况下，放在块里的内容将会被强行发送出去。下边的例子将会把发送的内容按照每块1KB的大小发送出去。

```
void setChunkedStreamingMode(int chunklen)
```

　　此方法用于在预先不知道内容长度时启用没有进行内部缓冲的 HTTP 请求正文的流。

 

例：

```java
httpConnection.setChunkedStreamingMode(1024);
```


## **2.9 缓存**

默认情况下，Java 1.5不返回任何数据。但通过派生java.net.ResponseCache类的子类，并安装为系统默认值，就可以创建自己的缓存。无论系统合适尝试通过协议处理器加载新的URL，它将首先在缓存中查找，若缓存找到了所需的内容，就不需要连接远程服务器了。若缓存数据不存在，则下载相应数据，并将其放入缓存中。