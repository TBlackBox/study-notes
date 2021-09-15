
# centos7安装java环境变量

1. 下载jdk包

* 官方下载
```
https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```
华为云镜像下载（较快）

```
https://repo.huaweicloud.com/java/jdk/
```

2. 创建文件夹并进入

```
mkdir /usr/local/java
```

3. 使用wget下载包
```
wget https://repo.huaweicloud.com/java/jdk/8u201-b09/jdk-8u201-linux-x
64.tar.gz
```

4. 解压文件
```
wget https://repo.huaweicloud.com/java/jdk/8u201-b09/jdk-8u201-linux-x
64.tar.gz
```

5. 环境变量配置
* 编辑好配置文件
```
export JAVA_HOME=/usr/local/java/jdk1.8.0_201
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

```

* 打开配置文件,将上面的内容追加到配置文件后面
```
vim /etc/profile
```

6. 使配置文件生效
```
source /etc/profile
```

7. 通命令测试是否安装好了
```
java -version
```

特别注意：
如果不没生效的话。可能由于下面可执行下面命令
```
yum install java-devel
```
原因： 跟rpm安装有关
