
# 重启
有人建议Kill all mysql。这种野蛮的方法其实是不行的，强制终止的话，如果造成表损坏，损失是巨大的。
安全的重启方法
```
$mysql_dir/bin/mysqladmin -u root -p shutdown
$mysql_dir/bin/safe_mysqld &
```
mysqladmin和mysqld_safe位于Mysql安装目录的bin目录下，很容易找到的。

## windows下重启MySQL服务

对于没装mysql图形管理端的用户来说启动和停止mysql服务：
 ```
  …\…\bin>net  stop mysql
  …\…\bin>net  start mysql
 ```

## linux 下重启Mysql服务

## 启动方式

1、使用 service 启动：service mysqld start

2、使用 mysqld 脚本启动：/etc/inint.d/mysqld start

3、使用 safe_mysqld 启动：safe_mysqld &

## 停止

1、使用 service 启动：service mysqld stop

2、使用 mysqld 脚本启动：/etc/inint.d/mysqld stop

3、mysqladmin shutdown

## 重启

1、使用 service 启动：service mysqld restart

2、使用 mysqld 脚本启动：/etc/inint.d/mysqld restart


# 查看链接数 状态
命令:show processlist
```
show processlist;
```
如果是root帐号，你能看到所有用户的当前连接。如果是其它普通帐号，只能看到自己占用的连接。
show processlist;只列出前100条，如果想全列出请使用`show full processlist`;
```
mysql> show full processlist;
```


命令: show status;
```
mysql>show status;
```
命令：`show status like '%下面变量%';`进行模糊查询。
```
Aborted_clients 由于客户没有正确关闭连接已经死掉，已经放弃的连接数量。
Aborted_connects 尝试已经失败的MySQL服务器的连接的次数。
Connections 试图连接MySQL服务器的次数。
Created_tmp_tables 当执行语句时，已经被创造了的隐含临时表的数量。
Delayed_insert_threads 正在使用的延迟插入处理器线程的数量。
Delayed_writes 用INSERT DELAYED写入的行数。
Delayed_errors 用INSERT DELAYED写入的发生某些错误(可能重复键值)的行数。
Flush_commands 执行FLUSH命令的次数。
Handler_delete 请求从一张表中删除行的次数。
Handler_read_first 请求读入表中第一行的次数。
Handler_read_key 请求数字基于键读行。
Handler_read_next 请求读入基于一个键的一行的次数。
Handler_read_rnd 请求读入基于一个固定位置的一行的次数。
Handler_update 请求更新表中一行的次数。
Handler_write 请求向表中插入一行的次数。
Key_blocks_used 用于关键字缓存的块的数量。
Key_read_requests 请求从缓存读入一个键值的次数。
Key_reads 从磁盘物理读入一个键值的次数。
Key_write_requests 请求将一个关键字块写入缓存次数。
Key_writes 将一个键值块物理写入磁盘的次数。
Max_used_connections 同时使用的连接的最大数目。
Not_flushed_key_blocks 在键缓存中已经改变但是还没被清空到磁盘上的键块。
Not_flushed_delayed_rows 在INSERT DELAY队列中等待写入的行的数量。
Open_tables 打开表的数量。
Open_files 打开文件的数量。
Open_streams 打开流的数量(主要用于日志记载）
Opened_tables 已经打开的表的数量。
Questions 发往服务器的查询的数量。
Slow_queries 要花超过long_query_time时间的查询数量。
Threads_connected 当前打开的连接的数量。
Threads_running 不在睡眠的线程数量。
Uptime 服务器工作了多少秒。
```

# 变量的显示和设置
# 显示变量
```
show variables;
```
同样支持模糊查询
```
show variables like "%max_con%";
```

# 设置变量 在.ini文件里面添加
*** 注意***
修改了my.cnf，需要重启MySQL服务。