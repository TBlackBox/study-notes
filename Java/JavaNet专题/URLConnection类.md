# 简介
URLConnection是一个抽象类，表示指向URL指定资源的活动连接。

URLConnection类本身依赖于Socket类实现网络连接。一般认为，URLConnection类提供了比Socket类更易于使用、更高级的网络连接抽象。但实际上，大多数程序员都会忽略它。因为URLConnection太贴近HTTP协议。它假定传输的内容前面都有MIME首部或类似的东西。但大多数协议（如FTP和SMTP）并不使用MIME首部。

## 继承树
Object
	URLConnection
		HttpURLConnection（java.net包下）
		JarURLConnection(java.net包下)
		URLConnection(sun.net.www包下)

URLConnection类声明为抽象类，除了connect()方法，其他方法都已经实现。



## 构造方法

URLConnection类的保护类型的构造方法：
```
protected URLConnection(URL url)
```
　　构造一个到指定 URL 的 URL 连接。

 

connect()方法由子类实现本地与服务器的连接方式。一般使用TCP socket，但也可以使用其他其他机制来建立。
```
abstract void connect()
```
打开到此 URL 引用的资源的通信链接（如果尚未建立这样的连接）。

 

当派生URLConnection子类时，通常会覆盖URLConnection的其他方法，使其返回有意义的值。

 

## **1.1 读取头部**
```
String getContentType()
```
　　返回 Content-type 头字段的值。即数据的MIME内容类型。若类型不可用，则返回null。

　　除了HTTP协议，极少协议会使用MIME首部。若内容类型是文本。则Content-type首部可能会包含一个标识内容编码方式的字符集。

例：Content-type:text/html; charset=UTF-8


```
int getContentLength()
```
　　返回 Content-length 头字段的值。该方法获取内容的字节数。许多服务器只有在传输二进制文件才发送Content-length首部，在传输文本文件时并不发送。若没有该首部，则返回-1。

　　若需要知道读取的具体字节数，或需要预先知道创建足够大的缓冲区来保存数据时，可以使用该方法。


```
String getContentEncoding()
```
　　返回 Content-encoding 头字段的值。获取内容的编码方式。若内容无编码，则该方法返回null。

　　注意：Content-encoding(内容编码)与字符编码不同。字符编码方式由Content-type首部或稳定内容的信息确定，它指出如何使用字节指定字符。内容编码方式则指出字节如何编码其他字节。

例：若编码格式为x-gzip，则可以使用java.util.zip.GZipInputStream来解码。


```
long getDate()
```
　　返回 date 头字段的值。获取请求的发送时间。即自1900年1月1日子夜后过去的毫秒数。

　　注意：这是从服务器的角度看到的发送时间。可能与本地时间不一致。若首部不包括Date字段，则返回0。


```
long getExpiration()
```
　　返回 expires 头字段的值。获取Expires的值。若无该字段，则返回0。0即表示不过期，永远缓存。

　　注意：该值是自1970年1月1日上午12:00后的毫秒数。


```
long getLastModified()
```
　　返回 last-modified 头字段的值。该值是自1900年1月1日子夜后过去的毫秒数。若无该字段，则返回0。

 

## **1.2 获取任意首部字段**
```
String getHeaderField(String name)
```
　　返回指定的头字段的值。


```
Map<String,List<String>> getHeaderFields()
```
　　返回头字段的不可修改的 Map。


```
String getHeaderFieldKey(int n)
```
　　返回第 n 个头字段的键。


```
String getHeaderField(int n)
```
　　返回第 n 个头字段的值。


```
long getHeaderFieldDate(String name, long Default)
```
　　返回解析为日期的指定字段的值。


```
int getHeaderFieldInt(String name, int Default)
```
　　返回解析为数字的指定字段的值。

 

 

## **1.3 配置连接**

URLConnection类的保护类型字段如下：
```
protected URL url
```
　　该字段指定URLConnection连接的URL。构造函数会在创建URLConnection时设置，此后就不能再改变。


```
protected boolean connected
```
　　当连接打开时，该字段为true。若连接关闭，则为false。没有直接读取或改变connected值的方法。任何导致URLConnection连接的方法都会将此变量设为true，包括connect()，getInputStream()，和getOutputStream()。

　　任何导致URLConnection断开的方法都会将此字段设为false。

　　java.net.URLConnection没有断开方法，但一些子类，如java.net.HttpURLConnection中存在这样的方法。


```
protected boolean allowUserInteraction
```
　　该字段表示是否允许与用户交互。默认为false，即不允许交互。该值只能在URLConnection连接前设置。

例：Web浏览器可能需要访问用户名和口令。


```
protected boolean doInput
```
　　该字段在URLConnection可用于输入时为true，否则为false。默认为true。


```
protected boolean doOutput
```
　　该字段在URLConnection可用于输出时为true，否则为false。默认为false。


```
protected long ifModifiedSince
```
　　该字段指示了将放置If-Modified-Since首部字段中的日期（格林威治标准时间1970年1月1日子夜后的毫秒数）。


```
protected boolean useCaches
```
　　该字段指定了是否可以在缓存可用时使用缓存。默认为true，表示缓存将被使用；false表示缓存不被使用。

 

以上字段定义了客户端如何向服务器作出请求。这些字段只能在URLConnection连接前修改。否则抛出IllegalStateException异常。

每个字段都有一些相应的方法。

- URL getURL()
- void setDoInput(boolean doinput)
- boolean getDoInput()
- void setDoOutput(boolean dooutput)
- boolean getDoOutput()
- void setUseCaches(boolean usecaches)
- boolean getUseCaches()
- void setAllowUserInteraction(boolean allowuserinteraction)
- boolean getAllowUserInteraction()
- void setIfModifiedSince(long ifmodifiedsince)
- long getIfModifiedSince()

还有一些方法，这些方法用于定义和获取URLConnection对象的默认行为：

- static boolean getDefaultAllowUserInteraction()
- boolean getDefaultUseCaches()
- void setDefaultUseCaches(boolean defaultusecaches)
- static void setDefaultAllowUserInteraction(boolean defaultallowuserinteraction)
- static void setFileNameMap(FileNameMap map)
- static FileNameMap getFileNameMap()

 

## **1.4 关于超时**

Java 1.5增加了如下4个方法，允许查询和修改连接的超时值。即底层socket在抛出SocketTimeoutException异常前等待远端响应时间。

设置获取读取的超时值，单位为毫秒。
```
int getReadTimeout()

void setReadTimeout(int timeout)
```
　　设置获取连接的超时值，单位为毫秒。


```
int getConnectTimeout()

void setConnectTimeout(int timeout)
```
　　这四个方法将0解释为永不超时。如果超时值为负数。两个设置方法都会抛出IllegalArgumentException异常。

 

 

## **1.5 配置HTTP首部**

客户端向服务器发送请求，不仅会发送请求行，还发送首部。Web服务器可以根据此信息向不同客户端提供不同的页面，获取和设置cookie，通过口令设置用户等等。通过在客户端发送和服务器响应的首部中放置不同的字段，就可以完成这些工作。

每个URLConnection的具体子类都在首部中设置一些默认的键值对（实际上只有HttpURLConnection这样做，因为HTTP是唯一以这种方式使用首部的主要协议。）

通过以下方法可以设置URLConnection在打开连接增加HTTP首部：
```
void setRequestProperty(String key, String value)
```
　　此方法只在连接打开之前打开，将会抛出IllegalStateException异常。但该方法仅支持对一个首部设置一个值。

 

HTTP允许一个熟悉有多个值。各个值之间用逗号隔开。

例：cookie:username=sjq; session=hdfu23asdjf901j;

 

使用如下方法为首部增加值：
```
void addRequestProperty(String key, String value)
```
　　该方法用于添加某个指定首部字段的值。

 

使用如下方法获取首部字段：
```
String getRequestProperty(String key)

Map<String,List<String>> getRequestProperties()
```
　　以上方法用于获取URLConneciton所拥有HTTP首部。

 

 

## **1.6 使用URLConnection与服务器交互**
```
InputStream getInputStream()
```
　　返回从此打开的连接读取的输入流。
```
OutputStream getOutputStream()
```
　　返回写入到此连接的输出流。

 

若只是向服务器请求数据，则为HTTP请求方法为GET。

若需要向服务器提交数据，必须在先调用setDoOutput(true)。当doOutput属性为true时，请求方法将由GET变为POST。

 

例：获URLConnection实例对象。

```
@Test
public void test(){
    try {
        URL url = new URL("http://www.baidu.com");
        URLConnection uc = url.openConnection();
    } catch (MalformedURLException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

 

例：使用URLConnection请求Baidu首页。

该程序请求方法为get。使用getInputStream()方法输出的内容只有响应体，不包含响应的首部。

```
@Test
public void testGetInputStream(){
    InputStream in = null;
    try {
        //连接
        URL url = new URL("http://www.baidu.com");
        URLConnection uc = url.openConnection();
         
        //获取读入流
        in = uc.getInputStream();
        //放入缓存流
        InputStream raw = new BufferedInputStream(in);
        //最后使用Reader接收
        Reader r = new InputStreamReader(raw);
         
        //打印输出
        int c;
        while((c = r.read())>0){
            System.out.print((char)c);
        }
         
    } catch (MalformedURLException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally{
        try {
            if(in!=null){
                in.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}   
```

 

 

## **1.6 获得内容**
```
Object getContent()
```
　　该方法实质上与URL类的getContent()方法相同。使用该方法JVM需要识别和理解该内容类型。该方法只用于支持类似HTTP的协议，协议需要对MIME内容类型有较清楚的了解。若内容未知，或协议无法理解内容类型，getContent()会抛出一个UnknowServiceException异常。
```
Object getContent(Class[] classes)
```
　　该方法可以选择将内容转换为哪个类对象，以提供数据的不同对象表示。该方法尝试以classes数组中某个类的形式返回内容。优先顺序就是数组的顺序。若任何一种请求类型都不可用，此方法会返回null。

 

 

## **1.7 ContentHandlerFactory**

URLConnection类包括一个静态的ContentHandler。即内容处理器。通过以下方法设置内容处理器工厂。
```
static void setContentHandlerFactory(ContentHandlerFactory fac)
```


## **1.8 URLConnection的安全**

建立网络连接、读写文件等等存在一些安全限制，URLConnection对象会受到这些安全限制的约束。
```
Permission getPermission()
```
　　该方法返回Permission对象，指出连接此URL所需的许可权。如果没有所需的许可权（即没有适当的安全管理器），它就会返回null。

 

 

## **1.9 MIME内容类型**
```
static String guessContentTypeFromName(String fname)
```
　　根据 URL 的指定 "file" 部分尝试确定对象的内容类型。
```
static String guessContentTypeFromStream(InputStream is)
```
　　根据输入流的开始字符尝试确定输入流的类型。

 

 

 

