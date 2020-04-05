

# httpsession的生命周期
1. 创建对象
+ 对jsp页面而言：客户端第一次访问，并且session的属性值为true,这会创建对象Httpsession对象，为false都不好创建Httpsession对象。
+ 对serlvet而言：客户短访问第一个WEB应用资源则只有调用`request.getSession()`或`request.getSession(true)`才会创建`HttpSession`对象。如果前面创建了，则可以通过session直接使用原来创建的。

2. 让session失效
+ 通过`httpSession.invalidate();`让session失效
+ 服务器卸载了当前web运用
+ 超过了过期时间会失效
```
//设置过期时间
void setMaxInactiveInterval(int var1);

//得到还有多久过期
int getMaxInactiveInterval();

```
也可用通往`web.xml`进行全局session的过期时间
```
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

*** 注意：并不是关闭了浏览器都关闭了session***