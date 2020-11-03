# 简介
MySQL删除数据的方式都有哪些？
常用的三种删除方式：通过 delete、truncate、drop 关键字进行删除；这三种都可以用来删除数据，但场景不同。

# delete、truncate、drop区别

## 执行速度区别
```
drop > truncate >> DELETE
```

## 原理区别

### DELETE

```
DELETE from tableName where xxx
```

**1、DELETE属于数据库DML操作语言**，只删除数据不删除表的结构，会走事务，执行时会触发trigger；

**2、在 InnoDB 中，DELETE其实并不会真的把数据删除，mysql 实际上只是给删除的数据打了个标记为已删除，因此 delete 删除表中的数据时，表文件在磁盘上所占空间不会变小，存储空间不会被释放，只是把删除的数据行设置为不可见。** 虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以重用这部分空间（重用 → 覆盖）。

**3、** DELETE执行时，会先将所删除数据缓存到rollback segement中，事务commit之后生效;

**4、** delete from table_name删除表的全部数据,对于MyISAM 会立刻释放磁盘空间，InnoDB 不会释放磁盘空间;

**5、对于delete from table_name where xxx 带条件的删除, 不管是InnoDB还是MyISAM都不会释放磁盘空间;**

**6、** delete操作以后使用 **optimize table table_name** 会立刻释放磁盘空间。不管是InnoDB还是MyISAM 。所以要想达到释放磁盘空间的目的，delete以后执行optimize table 操作。

**7、\**delete 操作是一行一行执行删除的，并且同时将该行的的\**删除操作日志记录在redo和undo**表空间中以便进行回滚（rollback）和重做操作，**生成的大量日志也会占用磁盘空间**。



### truncate

```
Truncate table tableName
```

**1、truncate：属于数据库DDL定义语言**，不走事务，原数据不放到 rollback segment 中，操作不触发 trigger。

**注意：执行后立即生效，无法找回 **

**2、** truncate table table_name **立刻释放磁盘空间 ，不管是 InnoDB和MyISAM** 。truncate table其实有点类似于drop table 然后creat,只不过这个create table 的过程做了优化，比如表结构文件之前已经有了等等。所以速度上应该是接近drop table的速度;

**3、truncate能够快速清空一个表。并且重置auto_increment的值。**

但对于不同的类型存储引擎需要注意的地方是：

- 对于MyISAM，truncate会重置auto_increment（自增序列）的值为1。而delete后表仍然保持auto_increment。
- 对于InnoDB，truncate会重置auto_increment的值为1。delete后表仍然保持auto_increment。但是在做delete整个表之后重启MySQL的话，则重启后的auto_increment会被置为1。

也就是说，InnoDB的表本身是无法持久保存auto_increment。delete表之后auto_increment仍然保存在内存，但是重启后就丢失了，只能从1开始。实质上重启后的auto_increment会从 SELECT 1+MAX(ai_col) FROM t 开始。

**4、** 小心使用 truncate，尤其没有备份的时候，如果误删除线上的表。

### drop

```
Drop table Tablename
```

**1、drop：属于数据库DDL定义语言，同Truncate；**

**执行后立即生效，无法找回 执行后立即生效，无法找回 执行后立即生效，无法找回**

**2、** drop table table_name 立刻释放磁盘空间 ，不管是 InnoDB 和 MyISAM; **drop 语句将删除表的结构被依赖的约束(constrain)、触发器(trigger)、索引(index); 依赖于该表的存储过程/函数将保留,但是变为 invalid 状态。**



# 总结
可以这么理解，一本书，delete是把目录撕了，truncate是把书的内容撕下来烧了，drop是把书烧了。最后提醒，删除需谨慎，在熟练的操作也请备好份。

