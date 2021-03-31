# 简介

gorm 是一个 go的orm库

## 具有如下特点

- 功能齐全的ORM
- 关联（具有一个，具有多个，属于，一对多，多态性，单表继承）
- 挂钩（创建/保存/更新/删除/查找之前/之后）
- 急于加载`Preload`，`Joins`
- 事务，嵌套事务，保存点，回滚到保存点
- 上下文，准备语句模式，DryRun模式
- 批处理插入，FindInBatches，使用地图查找/创建，使用SQL Expr和上下文评估器的CRUD
- SQL Builder，Upsert，锁定，优化器/索引/注释提示，命名参数，子查询
- 复合主键，索引，约束
- 自动迁移
- 记录仪
- 可扩展的灵活插件API：数据库解析器（多个数据库，读/写拆分）/ Prometheus…
- 每个功能都附带测试
- 开发人员友好





## 安装(国内访问很慢)

```
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite
```



