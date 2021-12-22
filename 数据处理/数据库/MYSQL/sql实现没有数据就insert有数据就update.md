# 简介
在向数据库里批量插入数据的时候，会遇到要将原有主键或者unique索引所在记录更新的情况，而如果没有主键或者unique索引冲突的时候，直接执行插入操作,下面有3总方式实现。

# 代码里直接判断
先通过条件查询出来判断,没有就`insert`,有数据就更新,毫无疑问，这是最笨的方法了，不断的查询判断，有主键或索引冲突，执行update,否则执行insert. 数据量稍微大一点这种方式就不行了。

# replace
这是mysql自身的一个语法，使用replace的时候。其语法为：
```
replace into tablename (f1, f2, f3) values(vf1, vf2, vf3),(vvf1, vvf2, vvf3)
```
这中语法会自动查询主键或索引冲突，如有冲突，他会先删除原有的数据记录，然后执行插入新的数据,这里可以批量插入。

# insert on duplicate key.
这也是一种方式，mysql的insert操作中也给了一种方式，语法如下：
```
INSERT INTO table (a,b,c) VALUES (1,2,3)
  ON DUPLICATE KEY UPDATE c=c+1;
```
在insert时判断是否已有主键或索引重复，如果有，一句update后面的表达式执行更新，否则，执行插入。

# 效率比对
+ 第一种方式不说了，比较笨

+ replace和insert　on duplicate key这两种方式

分析：
数据量少的时候，都快,数据量大的话，`insert　on duplicate key`都要快点了,因为`replace`要删除索引，维护索引成本比较高。