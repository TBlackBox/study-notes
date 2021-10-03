# 下载jdk
下载地址
```
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```
# 创建文件夹
```
mkdir /usr/local/java/
```
#  上传jar包
```
安装上传命令
yum install lrzsz

```

# 解压
```
rm -rf jdk-8u261-linux-x64.tar.gz 
```

如果是rpm安装,直接下面命令就可以了

```
rpm -ivh jdk-8u131-linux-x64.rpm
```

# 配置

```
 vi /etc/profile
```
 添加下面到最后
```
export JAVA_HOME=/usr/local/java/jdk1.8.0_261
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH


export JAVA_HOME=/usr/local/java/jdk1.8.0_261
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

# 是环境变量生效
```
source /etc/profile
```

# 测试是否配置完成
```
java -version
```



# 安装openjdk
## CentOS自带JDK是否已安装
```
yum list installed |grep java
```

## 查看yum库中的Java安装包
```
yum -y list java*
```

## 安装
```
yum -y install java-1.8.0-openjdk*
```

