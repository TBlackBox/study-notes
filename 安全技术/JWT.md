# 简介

### JSON Web Token是什么

> JSON Web Token (JWT)是一个开放标准(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

[官方介绍](https://jwt.io/introduction)

# 应用场景

- Authorization (授权) : 这是使用JWT的最常见场景。一旦用户登录，后续每个请求都将包含JWT，允许用户访问该令牌允许的路由、服务和资源。单点登录是现在广泛使用的JWT的一个特性，因为它的开销很小，并且可以轻松地跨域使用。
- Information Exchange (信息交换) : 对于安全的在各方之间传输信息而言，JSON Web Tokens无疑是一种很好的方式。因为JWT可以被签名，例如，用公钥/私钥对，你可以确定发送人就是它们所说的那个人。另外，由于签名是使用头和有效负载计算的，您还可以验证内容没有被篡改。

# JWT的结构

有三部分组成,中间用`.`连接：

- Header
- Payload
- Signature

一个典型的jwt像这个样子

````
xxxxx.yyyy.zzzz
````

## Header部分

header 典型的由两部分组成：

- token的类型("jwt")
- 算法名称(比如：HMAC  SHA256和RSA等)

例如

```json
{
    'alg': "HS256",
    'typ': "JWT"
}
```

然后用base64对这个json 编码就得到了JWT的第一部分header



## Payload部分

JWT的第二部分是payload，它包含声明（要求）。声明是关于实体(通常是用户)和其他数据的声明。

声明有三种类型: registered, public 和 private。

- Registered claims : 这里有一组预定义的声明，它们不是强制的，但是推荐。比如：iss (issuer), exp (expiration time), sub (subject), aud (audience)等。
- Public claims : 可以随意定义。
- Private claims : 用于在同意使用它们的各方之间共享信息，并且不是注册的或公开的声明。 

例如

```json
{
    "sub": '1234567890',
    "name": 'john',
    "admin":true
}
```

对payload进行Base64编码就得到JWT的第二部分

**注意：不要在JWT的payload或header中放置敏感信息，除非它们是加密的。**



## Signature部分
为了得到签名部分，你必须有编码过的header、编码过的payload、一个秘钥，签名算法是header中指定的那个，然对它们签名即可。

```java
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

签名是用于验证消息在传递过程中有没有被更改，并且，对于使用私钥签名的token，它还可以验证JWT的发送方是否为它所称的发送方。



# JWT如何工作

jwt生成的token通常放到header里面或者Bearer schema里

```
Authorization: Bearer
```

流程

```
用户携带用户名和密码请求访问 -服务器校验用户凭据 -应用提供一个token给客户端 -客户端存储token，并且在随后的每一次请求中都带着它 -服务器校验token并返回数据
```



# 基于服务器的身份认证

HTTP协议是无状态的，也就是说，如果我们已经认证了一个用户，那么他下一次请求的时候，服务器不知道我是谁，我们必须再次认证。

传统的做法是将已经认证过的用户信息存储在服务器上，比如Session。用户下次请求的时候带着Session ID，然后服务器以此检查用户是否认证过。

这种基于服务器的身份认证方式存在一些问题：

- Sessions : 每次用户认证通过以后，服务器需要创建一条记录保存用户信息，通常是在内存中，随着认证通过的用户越来越多，服务器的在这里的开销就会越来越大。
- Scalability : 由于Session是在内存中的，这就带来一些扩展性的问题。
- CORS : 当我们想要扩展我们的应用，让我们的数据被多个移动设备使用时，我们必须考虑跨资源共享问题。当使用AJAX调用从另一个域名下获取资源时，我们可能会遇到禁止请求的问题。
- CSRF : 用户很容易受到CSRF攻击。



#  token 和session的区别

1. Session是在服务器端的，而JWT是在客户端的。

2. Session方式存储用户信息的最大问题在于要占用大量服务器内存，增加服务器的开销。

而JWT方式将用户状态分散到了客户端中，可以明显减轻服务端的内存压力。

3. Session的状态是存储在服务器端，客户端只有session id；而Token的状态是存储在客户端。
4. 基于Token的身份认证是如何工作的 基于Token的身份认证是无状态的，服务器或者Session中不会存储任何用户信息。



**注意：每一次请求都需要token -Token应该放在请求header中 -我们还需要将服务器设置为接受来自所有域的请求，用Access-Control-Allow-Origin: ***



# jwt的优点

- 无状态和可扩展性：Tokens存储在客户端。完全无状态，可扩展。我们的负载均衡器可以将用户传递到任意服务器，因为在任何地方都没有状态或会话信息。 
- -安全：Token不是Cookie。（The token, not a cookie.）每次请求的时候Token都会被发送。而且，由于没有Cookie被发送，还有助于防止CSRF攻击。即使在你的实现中将token存储到客户端的Cookie中，这个Cookie也只是一种存储机制，而非身份认证机制。没有基于会话的信息可以操作，因为我们没有会话!

- oken在一段时间以后会过期，这个时候用户需要重新登录。这有助于我们保持安全。还有一个概念叫**token撤销**，它允许我们根据相同的授权许可使特定的token甚至一组token无效。



# JWT与OAuth的区别

- OAuth2是一种授权框架 ，JWT是一种认证协议 
- 无论使用哪种方式切记用HTTPS来保证数据的安全性 
- OAuth2用在使用第三方账号登录的情况(比如使用weibo, qq, github登录某个app)，而JWT是用在前后端分离, 需要简单的对后台API进行保护时使用。





# go中使用jwt

- [gin-jwt](https://github.com/appleboy/gin-jwt) - 用于Gin框架的JWT中间件
- [jwt-go](https://github.com/dgrijalva/jwt-go) 也是用于go jwt的中间件












