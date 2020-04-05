
# mysql 不会缓存的情况
1. 如果两个查询请求在任何字符上的不同（例如：空格、注释、大小写）
2. 如果查询请求中包含某些系统函数、用户自定义变量和函数、一些系统表
3. 涉及到 mysql 、information_schema、 performance_schema 数据库中的表，那这个请求就不会被缓存


# 缓存失效的情况
MySQL的缓存系统会监测涉及到的每张表，只要该表的结构或者数据被修改，如对该表使用了INSERT、 UPDATE、DELETE、TRUNCATE TABLE、ALTER TABLE、DROP TABLE或 DROP DATABASE语句，那使用该表的所有高速缓存查询都将变为无效并从高速缓存中删除！

# 查看服务器支持哪些存储引擎
```
SHOW ENGINES;
```

```
mysql> SHOW ENGINES;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)

mysql>
```

Support列表示该存储引擎是否可用，DEFAULT值代表是当前服务器程序的默认存储引擎。Comment列是对存储引擎的一个描述，英文的，将就着看吧。Transactions列代表该存储引擎是否支持事务处理。XA列代表着该存储引擎是否支持分布式事务。Savepoints代表着该存储引擎是否支持部分事务回滚。


# 设置表的存储引擎
```
CREATE TABLE 表名(
    建表语句;
) ENGINE = 存储引擎名称;
```

# 更改表的存储引擎
```
ALTER TABLE 表名 ENGINE = 存储引擎名称;
```

# 查看表的存储引擎
```
SHOW CREATE TABLE 表名\G
```


# 查看系统变量
```
SHOW VARIABLES [LIKE 匹配的模式];
```
例如：

```
//查看最大的链接数
SHOW VARIABLES like 'max_connections';
//模糊查询
SHOW VARIABLES LIKE 'default%';
```

注意：
上面查询的是的session作用范围的系统变量,等价于下面查询语句。
```
SHOW SESSION VARIABLES [LIKE 匹配的模式];
```
如果要查询全局的变量，使用下面的语句
```
SHOW GLOBAL VARIABLES [LIKE 匹配的模式];
```


# 设置系统变量
1. 启动的时候设置
```
mysqld --default-storage-engine=MyISAM --max-connections=10
```

2. 配置文件里面设置
```
[server]
default-storage-engine=MyISAM
max-connections=10
```
对于启动选项来说，如果启动选项名由多个单词组成，各个单词之间用短划线-或者下划线_连接起来都可以，但是它对应的系统变量的单词之间必须使用下划线_连接起来。

3. 设置全局作用的系统变量
```
语句一：SET GLOBAL default_storage_engine = MyISAM;
语句二：SET @@GLOBAL.default_storage_engine = MyISAM;
```

4. 设置会话作用的系统变量
```
语句一：SET SESSION default_storage_engine = MyISAM;
语句二：SET @@SESSION.default_storage_engine = MyISAM;
语句三：SET default_storage_engine = MyISAM;
```

# 查询状态变量
```
SHOW [GLOBAL|SESSION] STATUS [LIKE 匹配的模式];
```

例如：
```
SHOW STATUS LIKE 'thread%';
```

# 查看mysql 中字符集和比较规则的方式
```
SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];
SHOW COLLATION [LIKE 匹配的模式];
```

# 聚簇索引

1. 页内的记录是按照主键的大小顺序排成一个单向链表。各个存放用户记录的页也是根据页中用户记录的主键大小顺序排成一个双向链表。
2. 存放目录项记录的页分为不同的层次，在同一层次中的页也是根据页中目录项记录的主键大小顺序排成一个双向链表。B+树的叶子节点存储的是完整的用户记录。所谓完整的用户记录，就是指这个记录中存储了所有列的值（包括隐藏列）。


我们把具有这两种特性的B+树称为聚簇索引，所有完整的用户记录都存放在这个聚簇索引的叶子节点处。这种聚簇索引并不需要我们在MySQL语句中显式的使用INDEX语句去创建，InnoDB存储引擎会自动的为我们创建聚簇索引。另外有趣的一点是，在InnoDB存储引擎中，聚簇索引就是数据的存储方式（所有的用户记录都存储在了叶子节点），也就是所谓的索引即数据，数据即索引。

# 创建索引
方式1：
```
CREATE TALBE 表名 (
    各种列的信息 ··· , 
    [KEY|INDEX] 索引名 (需要被索引的单个列或多个列)
)
```
方式2：
```
ALTER TABLE 表名 ADD [INDEX|KEY] 索引名 (需要被索引的单个列或多个列);
```

# 删除索引
```
ALTER TABLE 表名 DROP [INDEX|KEY] 索引名;
```

# 创建联合索引
```
CREATE TABLE index_demo(
    c1 INT,
    c2 INT,
    c3 CHAR(1),
    PRIMARY KEY(c1),
    INDEX idx_c2_c3 (c2, c3)
);
```
在这个建表语句中我们创建的索引名是idx_c2_c3，这个名称可以随便起，不过我们还是建议以idx_为前缀，后边跟着需要建立索引的列名，多个列名之间用下划线_分隔开。

# 删除联合索引
```
ALTER TABLE index_demo DROP INDEX idx_c2_c3;
```

# 内连接与外连接
对于内连接的两个表，驱动表中的记录在被驱动表中找不到匹配的记录，该记录不会加入到最后的结果集，我们上边提到的连接都是所谓的内连接。

对于外连接的两个表，驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集。

在MySQL中，根据选取驱动表的不同，外连接仍然可以细分为2种：

左外连接

选取左侧的表为驱动表。

右外连接

选取右侧的表为驱动表。


# 子查询
## 子查询的语法条件
1. 子查询必须用() 括起来
2. `SELECT`里的子查询必须是标量子查询(即查询的结果只有一条)
3. 在想要得到标量子查询或者行子查询，但又不能保证子查询的结果集只有一条记录时，应该使用LIMIT 1语句来限制记录数量。
4. 对于[NOT] IN/ANY/SOME/ALL子查询来说，子查询中不允许有LIMIT语句。
5. 不允许在一条语句中增删改某个表的记录时同时还对该表进行子查询。
```
DELETE FROM t1 WHERE m1 < (SELECT MAX(m1) FROM t1);
```
6. 多余的情况
* ORDER BY子句
子查询的结果其实就相当于一个集合，集合里的值排不排序一点儿都不重要，比如下边这个语句中的ORDER BY子句简直就是画蛇添足：
```
SELECT * FROM t1 WHERE m1 IN (SELECT m2 FROM t2 ORDER BY m2);
```

* DISTINCT语句
集合里的值去不去重也没啥意义，比如这样：
```
SELECT * FROM t1 WHERE m1 IN (SELECT DISTINCT m2 FROM t2);
```

* 没有聚集函数以及HAVING子句的GROUP BY子句。
在没有聚集函数以及HAVING子句时，GROUP BY子句就是个摆设，比如这样：
```
SELECT * FROM t1 WHERE m1 IN (SELECT m2 FROM t2 GROUP BY m2);
```


# 剖析单条查询

## profiling的使用 
1. 可以通过下面语句开启sql语句执行记录，默认是关闭。
```
set profiling = 1;
```

2. 显示SQL语句执行的时间，比较简单。
```
show profiles;
```
3. 比较详细的显示查询的信息
```
show profile for query 2;
```
4. 通过下面语句进行关闭。
```
set profiling = 0;
```
*** 注意：****如果要通过排序显示的话，看mysql高性能83页，如果要查看sql语句的执行计划的话，可以使用`explain` 。