# 简介
表示一个统一资源标识符 (URI) 引用。

java用java.net.URI类表示
```
public final class URI
    implements Comparable<URI>, Serializable
```

此类的实例代表一个 URI 引用。此类提供了用于从其组成部分或通过解析其字符串形式创建 URI 实例的构造方法、用于访问实例的各个不同组成部分的方法，以及用于对 URI 实例进行规范化、解析和相对化的方法。此类的实例不可变。


# url的组成部分
```
[<scheme>:]<scheme-specific-part>[#<fragment>]
```
其中，方括号 [...] 用于描述可选组成部分，字符 : 和 # 代表它们自身。

绝对 URI 指定了scheme，非绝对的 URI 称为相对 URI。

URI 还可以根据其是否为不透明的 或分层的 进行分类。

### 不透明URI
不透明 URI 为绝对 URI，其特定于scheme的部分不是以斜线字符 '/' 开始。不透明 URI 无法进行进一步解析。
```
mailto:java-net@java.sun.com	
news:comp.lang.java
urn:isbn:096139210x
```
### 分层 URI
分层 URI 或者为绝对 URI（其特定于scheme的部分以斜线字符 '/' 开始），或者为相对 URI，即不指定scheme的 URI。
```
http://java.sun.com/j2se/1.3/ 
docs/guide/collections/designfaq.html#28 
../../../demo/jfc/SwingSet2/src/SwingSet2.java 
file:///~/calendar
```
分层 URI 还要按照下面的语法进行进一步的解析
```
[ scheme :][ // authority][ path][ ? query][ # fragment]
```
其中， :、 /、 ? 和 # 代表它们自身。
分层 URI 的特定于scheme的部分包含scheme和fragment部分之间的字符。
分层 URI 的授权组成部分（如果指定）为基于服务器的 或基于注册表的。基于服务器的授权按照如下众所周知的语法进行解析：
```
[ user-info @] host[ : port]
```
其中，字符 @ 和 : 代表它们自身。
几乎当前使用的所有 URI scheme都是基于服务器的。不能采用这种方式解析的授权组成部分被视为基于注册表的。

如果分层 URI 的路径组成部分以斜线字符 ('/') 开始，则称此 URI 本身为绝对的；否则它为相对的。分层 URI 或者为绝对的，或者指定了授权的路径，它始终为绝对的。

## 如上所述，URI 实例具有以下九个组成部分：
```
组成部分	类型
方案Scheme	String
特定于方案的部分SchemeSpecificPart	String
授权Authority	String
用户信息UserInfo	String
主机Host	String
端口Port	int
路径Path	String
查询Query  String
片段Fragment	String 
```
在给定实例中，任何特殊组成部分都或者为 未定义的，或者为 已定义的，并且有不同的值。未定义的字符串组成部分由 null 表示，未定义的整数组成部分由 -1 表示。已定义的字符串组成部分的值可以为空字符串；这与未定义的组成部分不等效。

实例中特定的组成部分是已定义的还是未定义的取决于所代表的 URI 类型。

- 绝对 URI 具有scheme组成部分。不透明的 URI 具有一个scheme、一个特定于scheme的部分，以及可能会有一个fragment，但是没有其他组成部分。
- 分层 URI 总是有一个路径（尽管可能为空）和一个特定于scheme的部分（它至少包含一个路径），还可以包含任何其他组成部分。
- 如果有授权组成部分且它又是基于服务器的，则主机组成部分将被定义，也有可能定义用户信息和端口组成部分。

# URI 实例的运算
此类支持的主要运算有 `规范化`、` 解析` 和 `相对化` 运算。
1、规范化 是将分层 URI 的路径组成部分中的不必要的 "." 和 ".." 部分移除的过程。每个 "." 部分都将被移除。".." 部分也被移除，除非它前面有一个非 ".." 部分。规范化对不透明 URI 不产生任何效果。

2、解析 是根据另一个基本 URI 解析某个 URI 的过程。得到的 URI 由两个 URI 组成部分构造，构造方式由 RFC 2396 指定，从基本 URI 取出原始 URI 中未指定的组成部分。对于分层 URI，原始的路径根据基本路径进行解析，然后进行规范化。

例如，解析 URI-1【docs/guide/collections/designfaq.html#28】
根据基本 URI【http://java.sun.com/j2se/1.3/】解析，结果为 URI
```
http://java.sun.com/j2se/1.3/docs/guide/collections/designfaq.html#28
```
解析相对 URI-2【../../../demo/jfc/SwingSet2/src/SwingSet2.java】
根据此结果应生成
```
http://java.sun.com/j2se/1.3/demo/jfc/SwingSet2/src/SwingSet2.java
```
支持对绝对和相对 URI，以及分层 URI 的绝对和相对路径的解析。根据任何其他 URI 对 URI file:///~calendar 进行解析只能生成原始的 URI，因为它是绝对路径。根据相对基础 URI-1 解析相对 URI-2 将生成规范的但依然是相对的 URI-2【demo/jfc/SwingSet2/src/SwingSet2.java】

3、相对化 是解析的逆过程：对于任何两个规范的 URI u 和 v，
```
u .relativize( u .resolve( v )).equals( v )  
u .resolve( u .relativize( v )).equals( v )。 
```
此运算在下面的场合非常有用：构造一个包含 URI 的文档，该 URI 必须尽可能是根据文档的基本 URI 建立的相对 URI。
例如，相对化 URI【http://java.sun.com/j2se/1.3/docs/guide/index.html】
根据基本 URI【http://java.sun.com/j2se/1.3】
生成了相对 URI【docs/guide/index.html】。



## 字符分类

RFC 2396 精确指出 URI 引用中的各个不同组成部分允许使用的字符。以下分类大部分取自该规范，这些约束的用法描述如下：

- alpha(初始的)	US-ASCII 字母字符，'A' 到 'Z' 以及 'a' 到 'z'
- digit	US-ASCII 十进制数字符，'0' 到 '9'
- alphanum	所有 alpha 和 digit 字符
- unreserved(不保留的)   所有 alphanum 字符及字符串 "_-!.~'()*" 中包含的字符
- punct	字符串 ",;:$&+=" 中包含的字符
- reserved(保留的)	所有 punct 字符及字符串 "?/[]@" 中包含的字符
- escaped(转义的)	转义八位组，即三部分组合：百分号 ('%') 后跟两个十六进制数（'0'-'9'、'A'-'F' 和 'a'-'f'）
- other	未包含在 US-ASCII 字符集中的 Unicode 字符不是控制字符（根据 Character.isISOControl 方法），并且不是空格字符（根据 Character.isSpaceChar 方法）（与 RFC 2396 有些出入，RFC 2396 限制为 US-ASCII）

全部合法 URI 字符集包含 unreserved、reserved、escaped 和 other 字符。



## 转义八位组、引用、编码和解码

RFC 2396 允许用户信息、路径、查询和fragment组成部分中包含转义八位组。转义在 URI 中实现两个目的：

- 当要求 URI 不能包含任何 other 字符以严格遵守 RFC 2396 时，需要对非 US-ASCII 字符进行编码。
- 要引用 组成部分中的非法字符。用户信息、路径、查询和fragment组成部分在判断哪些字符合法哪些字符非法上稍有不同。

在此类中由三个相关的运算实现了这两个目的：

- 字符的编码 方式是，用代表该字符在 UTF-8 字符集中的字符的转义八位组序列取代该字符。例如，欧元符号 ('\u20AC') 编码后为 "%E2%82%AC"。（与 RFC 2396 有些出入，RFC 2396 未指定任何特殊字符集）。
- 非法字符通过简单地对它进行编码来引用。例如，空格字符，用 "%20" 取代它来进行引用。UTF-8 包含 US-ASCII，因此对于 US-ASCII 字符，此转换具有的效果与 RFC 2396 的要求相同。
- 对转义八位组序列进行解码 的方法是，用它所代表的 UTF-8 字符集中的字符的序列将它取代。UTF-8 包含 US-ASCII，因此解码具有对引用的任何 US-ASCII 字符取消引用的效果，以及对任何编码的非 US-ASCII 字符进行解码的效果。如果在对转义八位组进行解码时出现解码错误，则出错的八位组用 Unicode 替换字符 '\uFFFD' 取代。

这些运算在此类的构造方法和方法中公开，如下所示：

- 单参数构造方法要求对参数中的任何非法字符都必须引用，并保留出现的任何转义八位组和 other 字符。
- 多参数构造方法根据其中出现的组成部分的需要对非法字符进行引用。百分号字符 ('%') 始终通过这些构造方法引用。任何 other 字符都将被保留。
- getRawUserInfo、getRawPath、getRawQuery、getRawFragment、getRawAuthority 和 getRawSchemeSpecificPart 方法以原始形式返回它们的相应组成部分的值，不解释任何转义八位组。由这些方法返回的字符串有可能包含转义八位组和 other 字符，但不包含任何非法字符。
- getUserInfo、getPath、getQuery、getFragment、getAuthority 和 getSchemeSpecificPart 方法解码相应的组成部分中的任何转义八位组。由这些方法返回的字符串有可能包含 other 字符和非法字符，但不包含任何转义八位组。
- toString 返回带所有必要引用的 URI 字符串，但它可能包含 other 字符。
- toASCIIString 方法返回不包含任何 other 字符的、完全引用的和经过编码的 URI 字符串。

## 标识

对于任何 URI u，下面的标识有效

```
new URI( u .toString()) .equals( u ) .
```

对于不包含冗余语法的任何 URI u，比如在一个空授权前面有两根斜线（如 file:///tmp/）和主机名后面跟一个冒号但没有端口（如 http://java.sun.com:），以及除必须引用的字符之外不对字符编码的情况，下面的标识也有效(3个参数)：

```
new URI( u .getScheme() , u .getSchemeSpecificPart() , u .getFragment()) .equals( u )
```

在所有情况下，以下标识有效(6个参数，草泥马的，根本没有6个参数的构造方法！)：

```
new URI( u .getScheme() , u .getUserInfo() , u .getAuthority() , u .getPath() , u .getQuery() , u .getFragment()) .equals( u )
```

如果 u 为分层的，则以下标识有效(7个参数)：

```
new URI( u .getScheme() , u .getUserInfo() , u .getHost() , u .getPort() , u .getPath() , u .getQuery() , u .getFragment()) .equals( u )
```

如果 u 为分层的并且没有授权或没有基于服务器的授权



## URI、URL 和 URN

URI 是统一资源 标识符，而 URL 是统一资源 定位符。因此，笼统地说，每个 URL 都是 URI，但不一定每个 URI 都是 URL。这是因为 URI 还包括一个子类，即统一资源 名称 (URN)，它命名资源但不指定如何定位资源。上面的 mailto、 news 和 isbn URI 都是 URN 的示例。

URI 和 URL 概念上的不同反映在此类和 URL 类的不同中。

此类的实例代表由 RFC 2396 定义的语法意义上的一个 URI 引用。URI 可以是绝对的，也可以是相对的。对 URI 字符串按照一般语法进行解析，不考虑它所指定的scheme（如果有）不对主机（如果有）执行查找，也不构造依赖于scheme的流处理程序。相等性、哈希计算以及比较都严格地根据实例的字符内容进行定义。换句话说，一个 URI 实例和一个支持语法意义上的、依赖于scheme的比较、规范化、解析和相对化计算的结构化字符串差不多。

作为对照，URL 类的实例代表了 URL 的语法组成部分以及访问它描述的资源所需的信息。URL 必须是绝对的，即它必须始终指定一个scheme。URL 字符串按照其scheme进行解析。通常会为 URL 建立一个流处理程序，实际上无法为未提供处理程序的scheme创建一个 URL 实例。相等性和哈希计算依赖于scheme和主机的 Internet 地址（如果有）；没有定义比较。换句话说，URL 是一个结构化字符串，它支持解析的语法运算以及查找主机和打开到指定资源的连接之类的网络 I/O 操作。



## URI的API

## 构造方法

- 1个参数，URI(String str)  通过解析给定的字符串构造一个 URI。
- 3个参数，URI(String scheme, String ssp, String fragment)   根据给定的组成部分构造 URI。
- 4个参数，URI(String scheme, String host, String path, String fragment)  根据给定的组成部分构造分层 URI。
- 5个参数，URI(String scheme, String authority, String path, String query, String fragment)  根据给定的组成部分构造分层 URI。
- 7个参数，URI(String scheme, String userInfo, String host, int port, String path, String query, String fragment)  根据给定的组成部分构造一个分层 URI。

## 静态方法

static URI create(String str)  通过解析给定的字符串创建 URI。 

- 此便捷工厂方法的工作方式类似于调用 URI(String) 构造方法；由该构造方法抛出的任何 URISyntaxException 都被捕获，并包装到一个新的 IllegalArgumentException 对象中，然后抛出此对象。
- 此方法的使用场合是：已知给定的字符串是合法的 URI（例如，程序中声明的 URI 常量），该字符串无法这样解析时将被视为编程错误。
- 当 URI 从用户输入或从其他易于引起错误的源构造时，应该使用直接抛出 URISyntaxException 的构造方法。

## get方法-1

对于如下get**()方法，URI 的**组成部分不能包含转义八位组，因此这些方法不执行任何解码操作。 

- String getScheme()  返回此 URI 的方案【Scheme】部分。 

- - URI 的方案组成部分（如果定义了）只包含 alphanum 类别和字符串 "-.+" 中的字符。
  - 方案始终以 alpha 字符开始。

- int getPort()  返回此 URI 的【端口号】。

- - URI 的端口组成部分（如果定义了）是一个非负整数。
  - 如果端口未定义，则返回 -1

- String getHost()  返回此 URI 的【主机】组成部分。 

## get方法-2

对于如下get**()方法，除了转义八位组的所有序列都已解码之外，其和对应的getRaw**()方法返回的字符串相等。

- String getRawPath()  返回此 URI 的原始/已解码的【路径】组成部分。

- - 如果路径未定义，则返回 null 

- String getRawAuthority()  返回此 URI 的原始/已解码的授权组成部分。 

- - URI 的授权组成部分（如果定义了）只包含“商用 at”字符 ('@') 和 unreserved、punct、escaped 和 other 类别中的字符。
  - 如果授权是基于服务器的，则它被进一步约束为具有有效的用户信息、主机和端口组成部分。

- String getRawFragment()  返回此 URI 的原始/已解码的片段组成部分。 

- String getRawQuery()  返回此 URI 的原始/已解码的查询组成部分。 

- String getRawSchemeSpecificPart()  返回此 URI 原始的特定于方案的解码部分（从不为 null）。 

- - 特定于方案的部分永远不会是未定义的，但它可能为空（""）。
  - URI 的特定于方案的部分只包含合法的 URI 字符。

- String getRawUserInfo()  返回此 URI 的原始/已解码的用户信息组成部分。

- - URI 的用户信息组成部分（如果定义了）只包含 unreserved、punct、escaped 和 other 类别中的字符。

## 其他方法

- boolean isAbsolute()  判断此 URI 是否为绝对的。 

- - 当且仅当 URI 具有方案组成部分时，它才是绝对的。

- boolean isOpaque()  判断此 URI 是否为不透明的。 

- - 当且仅当 URI 是绝对的且其特定于方案的部分不是以斜线字符 ('/') 开始时，此 URI 才是不透明的。
  - 不透明的 URI 具有一个方案、一个特定于方案的部分，以及可能会有的一个片段；所有其他组成部分都是未定义的。

- String toASCIIString()  以 US-ASCII 字符串形式返回此 URI 的内容。 

  - 如果此 URI 未包含 other 类别的任何字符，则调用此方法将返回的值与调用 toString 方法返回的值相同。否则，此方法的工作方式类似于调用该方法，然后对结果进行编码。

- URL toURL()  根据此 URI 构造一个 URL。 

  - 首先检查得知此 URI 为绝对路径后，此便捷方法的工作方式就好像调用它与对表达式 new URL(this.toString()) 进行计算是等效的。
  - 如果此 URL 不是绝对的，抛出 IllegalArgumentException；如果无法找到 URL 的协议处理程序，或者如果在构造 URL 时发生其他错误，抛出MalformedURLException 。

## 对URI的一些运算（规范化、相对化、解析）

- URI normalize()  规范化此 URI 的路径。 
- URI relativize(URI uri)  根据此 URI 将给定 URI 相对化。 
- URI parseServerAuthority()  尝试将此 URI 的授权组成部分（如果已定义）解析为用户信息、主机和端口组成部分。 
- URI resolve(URI uri)  根据此 URI 解析给定的 URI。 
- URI resolve(String str)  解析给定的字符串，然后在此 URI 的基础上构造一个新的 URI。 此方法与进行 resolve(URI.create(str)) 的作用相同。