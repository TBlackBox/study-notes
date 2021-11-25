

1. 按照下面的链接做
```
https://downloads.mariadb.org/mariadb/repositories/#distro=CentOS&distro_release=centos7-amd64--centos7&mirror=fder&version=10.2
```

2. 配置信息
```
# MariaDB 10.2 CentOS repository list - created 2019-09-27 08:35 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

```

3. 将上面的配置放到下面的配置文件里
```
vim /etc/yum.repos.d/MariaDB.repo
```

4. 执行安装命令
```
sudo yum install MariaDB-server MariaDB-client
```
5. 开启mysql
```
service mysql start
```

6. 链接mysql
```
mysql -uroot
```
(不需要密码)

 不行的话尝试：

 ```
 /usr/bin/mysqladmin -u root password '123456'
 mysql -uroot -p

 ```

 7. 创建数据库
 ```
create database blogdb default character set utf8mb4 collate utf8mb4_general_ci;
 ```

 8. 创建用户
 ```
 CREATE USER 'blog'@'%' IDENTIFIED BY '123456';
 ```

 9. 给新建的用户授权
 ```
grant all privileges on blogdb.* to blog@"%" identified by '123456';
 ```

 这样就可以连接了，记到开3306的端口哦





## 通过命令授权所有ip能访问

```sql
use mysql;
select host,user from user;
update user set host = '%' where user = 'root';
flush privileges;
```

