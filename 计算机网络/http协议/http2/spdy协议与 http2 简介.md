# SPDY 协议与 HTTP/2 简介

## 1.SPDY 协议

由于 HTTP/1.x 的缺陷，我们会引入雪碧图、将小图内联、使用多个域名等方式来提高性能，不过这些优化都绕开了协议。直到 2009 年，谷歌公开了自行研发的 SPDY 协议，主要解决 HTTP/1.1 效率不高的问题。谷歌推出 SPDY，才算是正式改造 HTTP 协议本身。降低延迟，压缩 header 等等，SPDY 的实践证明了这些优化的效果，也最终带来 HTTP/2 的诞生。
项目设定的目标：
- 页面加载时间 (PLT) 减少 50%。
- 无需网站作者修改任何内容。
- 将部署复杂性降至最低，无需变更网络基础设施。
- 与开源社区合作开发此新协议。
- 收集真实性能数据，验证实验性协议是否有效。

HTTP/1.1 有两个主要的缺点：**安全不足和性能不高。**由于背负着 HTTP/1.x 庞大的历史包袱, 所以协议的修改, 兼容性是首要考虑的目标，否则就会破坏互联网上无数现有的资产。SPDY 位于 HTTP 之下，TCP 和 SSL 之上，这样可以轻松兼容老版本的 HTTP 协议 (将 HTTP1.x 的内容封装成一种新的 frame 格式)，同时可以使用已有的 SSL 功能。

SPDY 协议在 Chrome 浏览器上证明可行以后，就被当作 HTTP/2 的基础，主要特性都在 HTTP/2 中得到继承。

## HTTP/2 简介

2015 年，HTTP/2 发布。HTTP/2 是现行 HTTP 协议（HTTP/1.x）的替代，但它不是重写，HTTP 方法 / 状态码 / 语义都与 HTTP/1.x 一样。**HTTP/2 基于 SPDY，专注于性能，最大的目标是在用户和网站间只用一个连接（connec-tion）**。从目前的情况来看，国内外一些排名靠前的站点基本都实现了 HTTP/2 的部署，使用 HTTP/2 能带来 20%~60% 的效率提升。

HTTP/2 由两个规范（Specification）组成：

1. Hypertext Transfer Protocol version 2 - RFC7540
2. HPACK - Header Compression for HTTP/2 - RFC7541