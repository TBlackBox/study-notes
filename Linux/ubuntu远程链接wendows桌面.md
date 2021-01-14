# 简介
有的时候，我们需要在ubuntu下面远程操作windows界面，可以通过下面的工具想来进行连接，就像windows远程桌面一样。
https://www.cnblogs.com/mawanglin2008/articles/3490906.html

## tsclient介绍

tsclient是图形化的，我没有使用，但其实tsclient功能也是很强大的，不仅支持RDP，还支持VNC，xdmcp协议。tsclient实际上是rdesktop的外壳程序，调用rdesktop进行连接。

 

tsclient官网：http://sourceforge.net/projects/tsclient/

tsclient安装方法：
```
apt-get install tsclient   //ubuntu/debian用户

apt-get install xnest      //xdmcp支持

apt-get install xtightvncviewer  //vnc支持

yum -y install tsclient   //fedora/centos用户
```
## rdesktop介绍