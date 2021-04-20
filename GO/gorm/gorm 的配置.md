# 配置
gorm 的配置分为全局配置和session配置
当然，全局针对全局会话，当seeeion 针单个配置



## 全局配置

```go
type Config struct {
	// GORM perform single create, update, delete operations in transactions by default to ensure database data integrity
	// You can disable it by setting `SkipDefaultTransaction` to true
	SkipDefaultTransaction bool  //GORM 会在事务里执行写入操作（创建、更新、删除） 跳过默认事务
	// NamingStrategy tables, columns naming strategy
	NamingStrategy schema.Namer //允许用户通过覆盖默认的NamingStrategy来更改命名约定，这需要实现接口 Namer 
	// FullSaveAssociations full save associations
	FullSaveAssociations bool
	// Logger
	Logger logger.Interface //允许通过覆盖此选项更改 GORM 的默认 logger
	// NowFunc the function to be used when creating a new timestamp
	NowFunc func() time.Time //更改创建时间使用的函数
	// DryRun generate sql without execute
	DryRun bool //生成 SQL 但不执行，可以用于准备或测试生成的 SQL
	// PrepareStmt executes the given query in cached statement
	PrepareStmt bool //PreparedStmt 在执行任何 SQL 时都会创建一个 prepared statement 并将其缓存，以提高后续的效率
	// DisableAutomaticPing
	DisableAutomaticPing bool //在完成初始化后，GORM 会自动 ping 数据库以检查数据库的可用性，若要禁用该特性，可将其设置为 true
	// DisableForeignKeyConstraintWhenMigrating
	DisableForeignKeyConstraintWhenMigrating bool //在 AutoMigrate 或 CreateTable 时，GORM 会自动创建外键约束，若要禁用该特性，可将其设置为 true
	// DisableNestedTransaction disable nested transaction
	DisableNestedTransaction bool
	// AllowGlobalUpdate allow global update
	AllowGlobalUpdate bool
	// QueryFields executes the SQL query with all fields of the table
	QueryFields bool
	// CreateBatchSize default create batch size
	CreateBatchSize int

	// ClauseBuilders clause builder
	ClauseBuilders map[string]clause.ClauseBuilder
	// ConnPool db conn pool
	ConnPool ConnPool
	// Dialector database dialector
	Dialector
	// Plugins registered plugins
	Plugins map[string]Plugin

	callbacks  *callbacks
	cacheStore *sync.Map
}
```



`NamingStrategy` 也提供了几个选项，如：

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  NamingStrategy: schema.NamingStrategy{
    TablePrefix: "t_",   // 表名前缀，`User`表为`t_users`
    SingularTable: true, // 使用单数表名，启用该选项后，`User` 表将是`user`
    NameReplacer: strings.NewReplacer("CID", "Cid"), // 在转为数据库名称之前，使用NameReplacer更改结构/字段名称。
  },
})
```

## session配置

```go
// Session session config when create session with Session() method
type Session struct {
	DryRun                   bool //生成sql 单不执行 用于测试和打印之类的
	PrepareStmt              bool //是否预编译 在执行任何 SQL 时都会创建一个 prepared statement 并将其缓存，以提高后续的效率
	NewDB                    bool //通过 NewDB 选项创建一个不带之前条件的新 DB
	SkipHooks                bool //跳过 钩子 方法，您可以使用 SkipHooks 会话模式
	SkipDefaultTransaction   bool //如果使用了Transaction ,跳过默认的事务，可以了理解为禁用事务
	DisableNestedTransaction bool //关闭嵌套事务支持
	AllowGlobalUpdate        bool //默认不允许进行全局 update/delete  设置true 可以启用全局更新
	FullSaveAssociations     bool //创建、更新记录时，GORM 会通过 Upsert 自动保存关联及其引用记录。 如果您想要更新关联的数据，您应该使用 FullSaveAssociations 模式
	QueryFields              bool //是否允许带属性查询，select * 和select 属性1，属性2 的区别
	Context                  context.Context //可以传入 Context 来追踪 SQL 操作
	Logger                   logger.Interface //使用 Logger 选项自定义内建 Logger
	NowFunc                  func() time.Time //允许改变 GORM 获取当前时间的实现
	CreateBatchSize          int   //批量插入的长度
}
```

参数可配置到全局和单给会话，只是作用范围不同，意思是一个。