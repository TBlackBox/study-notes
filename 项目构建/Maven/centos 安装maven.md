1. 下载maven管理包

   ```
wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz   
   ```
如果没有 wget的话，安装
```
yum -y install wget
```

2. 解压
```
tar -zxvf  apache-maven-3.6.3-bin.tar.gz
```

3. 改名字
```
mv apache-maven-3.6.3 maven
```

4. 配置环境变量
编辑`profile`文件
```
vim /etc/profile
```
加下面内容
```
export M2_HOME=/maven/maven                                   -- maven 安装的路径
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
```

5. 使配置文件生效
```
source /etc/profile
```

6. 查看是否正确安装
```
mvn  -v 
```