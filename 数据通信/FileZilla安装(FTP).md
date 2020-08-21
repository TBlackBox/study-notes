# 简介
在链接vindows server服务器的时候，如果要传输文件的话，通过复制和粘贴进行文件传输，但是经常不稳定，下面介绍通过FileZilla的安装方法，



# 安装FileZilla
## 下载FileZilla
下载地址
```
https://sourceforge.net/projects/filezilla/files/
```
客服端，服务端下载地址
```
Client:
https://filezilla-project.org/download.php?show_all=1
Server: 
https://filezilla-project.org/download.php?show_all=1&type=server
```

## 服务端安装
1. 阅读许可协议，点击【I Agree】进入下一步安装；选择安装内容，默认安装标准即可，点击【Next】（其中“Source Code”是源代码，不需要勾选）
2. 选择安装路径、选择FileZilla Server的启动方式以及管理端口。
共有3种启动方式：
- ileZilla Server作为服务安装，随Windows系统启动；
- 将FileZilla Server作为服务安装，手动启动；
- 不将FileZilla Server作为服务安装，随Windows系统启动。
一般情况选择第一种。管理端口选择未被占用的端口即可。
3. 配置控制台启动方式。
共有3种选择：
- 用户适用，自动启动；
- 仅对当前用户适用，自动启动；
- 手动启动。
一般情况选择第一种即可。点击【Install】开始安装。

4. 启动FileZilla Server，出现一个配置IP、管理端口的对话框，这里输入本地IP（即127.0.0.1），并且输入之前配置的管理端口（本例为14147），点击【OK】

5. 服务端就安装完成了，

6. 点击工具栏上的user按钮小图标，进入用户配置界面；点击【Add】按钮新增用户；在弹出的对话框中输入用户名（本例测试用户名为test,【OK】进入下一步
7. 勾选“password”，为新增的用户设置密码，点击【OK】按钮
8. 出现提示告知添加用户目录，点击【确定】进入设置界面。点击【Add】新增用户目录。
9. 选择要用作FTP资源的目录（本例使用已新建的test，点击【确定】
10. 对于每一个资源目录，选中能访问该目录的用户并配置相关的访问权限（请删除Shared Folders下面的New directory项，否则可能报错)。至此，FileZilla服务已经搭建好。

### 错误解决
#### Warning: FTP over TLS is not enabled, users cannot securely log in.
表示未启用tls 模式，启用即可
1. 进入服务端设置-> ssl/tls设置
2. 点击启用FTP在ssl/tls的支持
3. 填写相关信息，生成一个证书，服务器ip用机器ip即可（127.0.0.1）
4. 秘钥文件保存到一个目录中
5. 点击生成证书，一会儿就生成成功了


## 客服端安装
1. 下载客服端，安装即可
2. 客户端通过本地FileZilla工具，连接至云服务器上搭建的FTP服务器。输入FTP服务器公网IP、账号、密码，点击【快速连接】，即可看到服务器分享给该用户的目录
3. FileZilla服务器此刻可监控到客户端的连接


# 总结
参看地址
```
https://www.cnblogs.com/52why/p/6564902.html
```