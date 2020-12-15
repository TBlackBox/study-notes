# 简介
跨站脚本攻击（XSS）是客服端脚本安全中的头号大敌OWASP TOP 10多次把XSS列在榜首，这里简单的介绍他的原理和如何防御它。

跨站脚本攻击，Cross Site Script为了和CSS区分所以叫XSS。


XSS攻击指，攻击者往Web页面里插入恶意html代码，当其它用户浏览该页之时，嵌入其中Web里面的html代码会被执行，从而达到恶意攻击用户的目的。

XSS根据效果可以分成：

- 反射型XSS：简单把用户输入的数据反射给浏览器，例如诱使用户点击个恶意链接来达到攻击的目的
- 存储型XSS：把用户输入的数据存储到服务器，例如黑客发表包含恶意js代码的文章，发表后所有浏览文章的用户都会在他们的浏览器执行这段恶意代码
案例：

            2011年，新浪微博XSS蠕虫事件：攻击者利用广场的一个反射性XSS URL，自动发送微博、私信，私信内容又带有该XSS URL，导致病毒式传播。百度空间、twitter等SNS网站都发生过类似事件。

- DOM Based XSS
通过修改页面的DOM节点来形成XSS,称之为DOM Based XSS。


# XSS Payload
XSS 攻击成功后，攻击者能够对用户当前浏览的页面植入恶意脚本，控制用户的浏览器，这样用于完成具体功能的脚本，被称为“XSS Payload”；
这些恶意脚本可以用来获取用户的cookie,从而登录用的账户，还能都做下面的事情。

1. 构造GET和POST请求
2. XSS 钓鱼
- 通过XSS Payload够着的脚本，模仿源站的页面，让用户输入信息之类的或者是伪造一个登录框等等。
3. 识别用户浏览器
XSS可以读取浏览器的UserAgent对象
4. 识别与用户安装的软件
- 在IE中，可以通过判断ActiceX控件的classid是否存在，来检查用户是否安装了该软件。
5. CSS History Hack
- 通过Css 来发现用户曾经访问过得网站，主要是利用style的`visited`属性.
6. 获取用户的真实ip
- JavaScript没得获取本地IP地址的能力，可以借助三方软件来，例如
   -  如果安装了java环境，通过Java Applet的接口获取客服端的本地IP地址。
   - 还可以通过ActiveX控件查询本地IP地址。

   # XSS攻击平台
   XSS Payload如此强大，为了使用方便，有安全研究者就把许多功能封装起来，成文XSS攻击平台，这些攻击平台主要是演示XSS的危害，方便渗透测试用，下面介绍几个常用的XSS攻击平台。
   1. Attack API
   PHP的系统
   2. BeEF
   有控制台，在后端可以控制前端一切。
   3. XSS-Proxy
   
轻量级的，可以通过iframe的方式实时的远程控制被XSS攻击的浏览器。


# 调试JavaScript
1. Firebug
2. IE & Developer Tools
3. Fiddler
3. HttpWath

# XSS 构建技巧
## 利用字符编码
## 绕过长度限制
## 使用`<base>`标签
## 妙用window.name


# XSS的防御
1. 设置Httponly
JavaScript禁止访问带有HttpOnly属性的Cookie,可以针对不同的cookie设置。

2. 输入检查
对输入的时间进行检查，也是XSS Filter。

3. 输出检查
4. 安全的编码函数
编码分为很多种，针对html代码的编码方是HtmlEncode.

# 总结
这里只是简单介绍了一哈，在具体的运用中，还要根据具体的情况来做，比较安全要运用到实际问题中，写代码也需要工匠精神啊。

