# 简介
URL（Uniform Resource Locator）中文名为统一资源定位符，有时也被俗称为网页地址。表示为互联网上的资源，如网页或者FTP地址。

在java.net包下，通过URL类来表示URl.

## URL可以分为下面几部分
```
protocol://host:port/path?query#fragment
```
protocol(协议)可以是 HTTP、HTTPS、FTP 和 File，port 为端口号，path为文件路径及文件名。

### HTTP 协议的 URL 实例如下:
```
http://www.runoob.com/index.html?language=cn#j2se
```
URL 解析：
- 协议为(protocol)：http
- 主机为(host:port)：[www.runoob.com](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.runoob.com)
- 端口号为(port): 80 ，以上URL实例并未指定端口，因为 HTTP 协议默认的端口号为 80。
- 文件路径为(path)：/index.html
- 请求参数(query)：language=cn
- 定位位置(fragment)：j2se，定位到网页中 id 属性为 j2se 的 HTML 元素位置 。


# URL类的方法
在java.net包中定义了URL类，该类用来处理有关URL的内容。对于URL类的创建和使用，下面分别进行介绍。
```
public final class URL implements java.io.Serializable {}
```

java.net.URL提供了丰富的URL构建方式，并可以通过java.net.URL来获取资源。

| 序号 | 方法描述                                                     |
| ---- | ------------------------------------------------------------ |
| 1    | public URL(String protocol, String host, int port, String file) throws MalformedURLException. 通过给定的参数(协议、主机名、端口号、文件名)创建URL。 |
| 2    | public URL(String protocol, String host, String file) throws MalformedURLException 使用指定的协议、主机名、文件名创建URL，端口使用协议的默认端口。 |
| 3    | public URL(String url) throws MalformedURLException 通过给定的URL字符串创建URL |
| 4    | public URL(URL context, String url) throws MalformedURLException 使用基地址和相对URL创建 |



URL类中包含了很多方法用于访问URL的各个部分，具体方法及描述如下：

| 序号 | 方法描述                                                     |
| ---- | ------------------------------------------------------------ |
| 1    | public String getPath() 返回URL路径部分。                    |
| 2    | public String getQuery() 返回URL查询部分。                   |
| 3    | public String getAuthority() 获取此 URL 的授权部分。         |
| 4    | public int getPort()  返回URL端口部分。                      |
| 5    | public int getDefaultPort() 返回协议的默认端口号。           |
| 6    | public String getProtocol()返回URL的协议。                   |
| 7    | public String getHost() 返回URL的主机。                      |
| 8    | public String getFile() 返回URL文件名部分。                  |
| 9    | public String getRef() 获取此 URL 的锚点（也称为"引用"）。   |
| 10   | public URLConnection openConnection() throws IOException 打开一个URL连接，并运行客户端访问资源。 |



# openConnection()方法
```
 public URLConnection openConnection() throws java.io.IOException {
        return handler.openConnection(this);
    }
```
openConnection() 返回一个 java.net.URLConnection。

*** 注意 ***

- 你连接HTTP协议的URL, openConnection() 方法返回 HttpURLConnection 对象。
- 连接的URL为一个 JAR 文件, openConnection() 方法将返回 JarURLConnection 对象。



