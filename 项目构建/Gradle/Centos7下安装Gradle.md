# centos 7 下安装gradle
+ 创建文件夹
```
mkdir /opt/gradle
```

+ 下载
```
wget https://downloads.gradle-dn.com/distributions/gradle-5.6.1-all.zip
```

+ 解压 
```
unzip /opt/gradle gradle-5.6.1-all.zip
```

+ 配置 
```
vim /etc/profile
```
在底部添加 
```
    PATH=$PATH:/opt/gradle/gradle-5.6.1/bin
    export PATH
```
+ 让配置生效
```
source /etc/profile
```

+ 检查是否完成
```
gradle --version
```