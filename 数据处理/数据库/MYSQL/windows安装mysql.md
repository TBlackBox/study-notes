# windows 安装mysql免安装版教程

## 下载安装包
### 下载地址
```
https://dev.mysql.com/downloads/mysql/
```
### 选择版本
点击
```
Looking for the latest GA version?
```

## 解压到盘符
```
D:\mysql-5.7.31-winx64
```

### 接下来就是mysql的配置
首先添加path环境变量，路径指向mysql所在bin目录下
我的电脑->属性->高级系统设置->环境变量

- 添加环境变量
- 新建JAVA_HOME
```
MYSQL_HOME=D:\mysql-5.7.31-winx64
```
path添加如下：
```
%MYSQL_HOME%\bin;
```

### 在`MYSQL_HOME`目录下创建my.ini配置文件（也就是安装目录下）
配置下面内容
```properties
[client]
port=3306
default-character-set=utf8

[mysqld]
#解压目录
basedir=D:\mysql-5.7.31-winx64
#解压目录下data目录
datadir=D:\mysql-5.7.31-winx64\data
port=3306
character_set_server=utf8
#导出mysql数据的目录
secure_file_priv =D:\mysql-5.7.31-winx64\data
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
#开启查询缓存
explicit_defaults_for_timestamp=true
skip-grant-tables

[WinMySQLAdmin]
D:\mysql-5.7.31-winx64\bin\mysqld.exe
```
### 在根目录下创建data文件夹
这个时候我们的初始化配制文件已经写好了，但是由于mysql 5.7版本后已经不自带data，我们还没有默认数据库就是上文说的data文件夹。我们在管理员模式下运行命令行。并且打开到自己的解压缩目录下。
并取得管理员权限运行,mysqld在bin文件夹里
打开管理员模式cmd的，
进入mysql\bin目录下，
```
mysqld –initialize-insecure –user=mysql
```

安装
```
mysqld -install
```

启动服务
```
net start mysql
```
登录
```
mysql -u root -p
```
打开数据库

由于我们是第一次登陆mysql 直接回车键登录。
设置密码
```
set password for root@localhost = password('1234');
```
修改密码成功后输入quit退出mysql
然后再次登录：mysql -u root -p, 输入password 验证是否修改成功
若出现错误：
```
mysql access denied for user 'root'@'localhost' (using password: YES)
```
则在my.ini文件最后一行加入
```
skip-grant-tables
```
