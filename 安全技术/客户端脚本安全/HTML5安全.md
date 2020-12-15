# 简介
HTML5是W3C置顶的新一代HTML标准，这个标准还在不断的修改。但现在已经普及了。当然也存在一些安全问题，下面简单的介绍一下。

# 一些新的标签
## 新标签XSS
`<video>`,`<audio>`等需要加入XSS Filter黑名单避免发生XSS。

## iframe的新属性sendbox
`<iframe>`加载的内容被视为一个独立的源，其中脚本将禁止执行。可以通过参数来进行精确的控制:
- allow-same-origin:允许同源访问
- allow-top——navigtion: 允许访问顶层窗口。
- allow-forms： 允许提交表单
- allow-scripts：允许执行脚本

注意：弹框脚本是不允许的。

## Link Types: noreferrer
HTML5中为新的`<a>`,`<area>`定义了新的Link Types:noreferrer。
指定noreferrer后，浏览器在请求该标签指定的地址是将不在发送Referer。
```html
<a hret="www.baidu.com" rel="noreferrer">test</a>
```
包含了敏感信息和隐私。

## Cancas
可以让js 直接操作图片对象。直接操作像素，苟江图片区域。

*** 可以用来破解验证码（这个牛逼哦）***

# 其他安全问题
1. Cross-origin Resource Sharing 尽量不要管使用通配符`*`，使用个更多的HTTP Header做更精确的控制。
2. PostMessage 跨窗口传递消息
postMessage允许每一个windows对象向其他窗口发送文本消息不接受同源策略的限制。
*** 可能导致DOM based XSS 的产生 ***

# WEB Stoeage
# Storage类型
- Session Storage
- Local Storage

安全隐患
攻击者将恶意代码保存在Web Storage中，从而实现跨页面攻击。

# 总结
HTML5有许多新功能，但在使用新功能的同时，也要注意安全问题。
