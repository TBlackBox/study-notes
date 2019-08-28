
	# 下载windows压缩包,解压
		https://dev.mysql.com/downloads/mysql/

		* 可以点击: Looking for previous GA versions? 来下载以前的版本(5.7.x)
	
	# 添加环境变量
		MYSQL_HOME=D:\mysql-5.7.26-winx64
	
	# 添加PATH变量
		%MYSQL_HOME%\bin;

	# 创建ini配置文件
		* 在mysql的解压根目录创建文件:my.ini

[mysql]
default-character-set=utf8mb4 

[mysqld]
port = 3306 
basedir=D:\mysql\mysql-5.7.26-winx64
datadir=D:\mysql\mysql-5.7.26-winx64\data
max_connections=200
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[client]
default-character-set=utf8mb4


	
	# 初始化
		mysqld --initialize-insecure

		* 初始化无密码的root用户
		* 登录时输入密码直接回车
		* 登录成功后修改密码
			set password for 'root'@'%' = password('root');
	
	# 安装到系统服务
		mysqld --install

		* 使用系统命令维护
			net start mysql
			net stop mysql
		
		* 出错可以卸载重新安装
			mysqld --remove mysql

# 注意：
+ 5.7 更新密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourPassword';

+ mysql 8.0.11更新密码:
alter user 'root'@'localhost' IDENTIFIED BY 'YourPassword';

+ 允许数据库让外部ip访问
    grant all privileges on *.* to 你的数据库账号@'%'  identified by '你对应账号的密码'；
