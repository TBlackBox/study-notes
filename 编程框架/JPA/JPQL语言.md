# 简介
JPQL语言，即 Java Persistence Query Language 的简称。

JPQL 是一种和 SQL 非常类似的中间性和对象化查询语言，它最终会被编译成针对不同底层数据库的 SQL 查询，从而屏蔽不同数据库的差异。

JPQL语言的语句可以是 select 语句、update 语句或delete语句，它们都通过 Query 接口封装执行。

***注意：JPQL语言不支持insert语句！***