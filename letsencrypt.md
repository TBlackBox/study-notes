[文档](https://letsencrypt.org/zh-cn/how-it-works/)
https://www.jianshu.com/p/c5c9d071e395

在 VPS 建站下手动安装 SSL 证书时，如果出现此错误提示：
```
Problem binding to port 80: Could not bind to IPv4 or IPv6.
```
则原因是 nginx 占用了80端口，输入 service nginx stop。然后再次执行证书安装命令，即可顺利安装。

安装完毕后，输入 service nginx start，重启 nginx 服务。


阿里云域名快速安装
https://www.laozuo.org/11729.html