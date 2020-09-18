
# 官方网站

```
https://gradle.org/

```
# 官方安装教程
```
https://gradle.org/install/
```

# 通过SDKMAN

## 安装sdkman
windows安装和linux 安装里面都有介绍
### 官网地址
```
https://sdkman.io/install
```

安装命令
```
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk version
```

## 安装gralde
```
sdk install gradle 6.6.1
```
注意:6.6.1 是版本号，可以在官网上看。

# centos 7 下安装gradle

+ 创建文件夹
```
mkdir /opt/gradle
```

+ 下载
这里可以先通过下载，在上传解压安装和配置
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

