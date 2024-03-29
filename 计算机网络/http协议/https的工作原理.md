# 简介
HTTPS在传输数据之前需要客户端（浏览器）与服务端（网站）之间进行一次握手，在握手过程中将确立双方加密传输数据的密码信息。

TLS/SSL协议不仅仅是一套加密传输的协议，更是一件经过艺术家精心设计的艺术品，*** TLS/SSL中使用了非对称加密，对称加密以及HASH算法***。

# https 的握手过程
1. 浏览器将自己支持的一套加密规则发送给网站。
2. 网站从中选出一组加密算法与HASH算法，并将自己的身份信息以证书的形式发回给浏览器。证书里面包含了网站地址，加密公钥，以及证书的颁发机构等信息。
3. 浏览器获得网站证书之后浏览器要做以下工作：
    - 验证证书的合法性（颁发证书的机构是否合法，证书中包含的网站地址是否与正在访问的地址一致等），如果证书受信任，则浏览器栏里面会显示一个小锁头，否则会给出证书不受信的提示。
    - 如果证书受信任，或者是用户接受了不受信的证书，浏览器会生成一串随机数的密码，并用证书中提供的公钥加密。
    - 使用约定好的HASH算法计算握手消息，并使用生成的随机数对消息进行加密，最后将之前生成的所有信息发送给网站。
4. 网站接收浏览器发来的数据之后要做以下的操作：
    - 使用自己的私钥将信息解密取出密码，使用密码解密浏览器发来的握手消息，并验证HASH是否与浏览器发来的一致。
    - 使用密码加密一段握手消息，发送给浏览器。
5.浏览器解密并计算握手消息的HASH，如果与服务端发来的HASH一致，此时握手过程结束，**之后所有的通信数据将由之前浏览器生成的随机密码并利用对称加密算法进行加密**。

这里浏览器与网站互相发送加密的握手消息并验证，目的是为了保证双方都获得了一致的密码，并且可以正常的加密解密数据，为后续真正数据的传输做一次测试。另外，HTTPS一般使用的加密与HASH算法如下：

      非对称加密算法：RSA，DSA/DSS
      对称加密算法：AES，RC4，3DES
      HASH算法：MD5，SHA1，SHA256

 # HTTPS对应的通信时序图

[![image](../../sources/img/https.png)



# HTTPS协议和HTTP协议的区别
 1. https协议需要到ca申请证书，一般免费证书很少，需要交费。 
2. http是超文本传输协议，信息是明文传输，https 则是具有安全性的ssl加密传输协议。 
3. http和https使用的是完全不同的连接方式用的端口也不一样,前者是80,后者是443。
4. http的连接很简单,是无状态的 。
5. HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议， 要比http协议安全。