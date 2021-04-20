# gorm 的整体结构



grom框架 大概的一个实现思路

1. 通过`Dialector`和gorm里面的一个Config配置创建一个db的结构体.(不同的数据库 dialector 不一样，当然配置和连接也不一样)。
2. db 可以做一个全局的配置使用，db可以创建会话(session:创建带配置的新建会话模式)和statement(声明或者陈述,也就是sql语句)
3. statement是有由clause(条款,可以理解为sql的每个部分）组装出来。
4. 在通过终止方法，执行sqlj即可。



# 链式操作

链式操作，有起始，中间链，和终止，gorm 通过链式的形式对生成sql



## 创建会话方法

在初始化了 `*gorm.DB` 或 `新建会话方法` 后， 调用下面的方法会创建一个新的 `Statement` 实例而不是使用当前的

GORM 定义了 `Session`、`WithContext`、`Debug` 方法做为 `新建会话方法`



## 链式方法

在`chainable_api.go`中

链式方法是将 `Clauses` 修改或添加到当前 `Statement` 的方法，例如：

```
`Where`、`Select`、`Omit`、`Joins`、`Scopes`、`Preload`、`Raw`（但在构建 SQL 语句时，`Raw` 不能与其它链式方法一起使用）
```



## 完成的方法

在`finisher_api.go`中

Finishers 是会立即执行注册回调的方法，然后生成并执行 SQL，比如这些方法：

```
Create`, `First`, `Find`, `Take`, `Save`, `Update`, `Delete`, `Scan`, `Row`, `Rows
```



#  注意会话的安全问题

新建statement,才不会导致复用。