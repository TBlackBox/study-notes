# 模型

也就是go里面的结构体，由 Go 的基本数据类型、实现了 [Scanner](https://pkg.go.dev/database/sql/?tab=doc#Scanner) 和 [Valuer](https://pkg.go.dev/database/sql/driver#Valuer) 接口的自定义类型及其指针或别名组成

例如

```go
type User struct {
  ID           uint
  Name         string
  Email        *string
  Age          uint8
  Birthday     *time.Time
  MemberNumber sql.NullString
  ActivedAt    sql.NullTime
  CreatedAt    time.Time
  UpdatedAt    time.Time
}
```

gorm 倾向于约定大于配置。默认情况下：

1. ID 作为主键
2. 结构体名作为表名，通过蛇形命名法命名(即单词间通过下划线连接如：user_name)
3. 字段名作为列名（也是蛇形命名法）
4. `CreatedAt`、`UpdatedAt` 字段追踪创建、更新时间

# gorm.Model
GORM 定义一个 gorm.Model 结构体，其包括字段 ID、CreatedAt、UpdatedAt、DeletedAt

```go
// gorm.Model 的定义
type Model struct {
  ID        uint           `gorm:"primaryKey"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt gorm.DeletedAt `gorm:"index"`
}
```

您可以将它嵌入到您的结构体中，可以减少代码的字段。

### 字段级权限控制

可导出的字段在使用 GORM 进行 CRUD 时拥有全部的权限，此外，GORM 允许您用标签控制字段级别的权限。这样您就可以让一个字段的权限是只读、只写、只创建、只更新或者被忽略

```go
type User struct {
  Name string `gorm:"<-:create"` // 允许读和创建
  Name string `gorm:"<-:update"` // 允许读和更新
  Name string `gorm:"<-"`        // 允许读和写（创建和更新）
  Name string `gorm:"<-:false"`  // 允许读，禁止写
  Name string `gorm:"->"`        // 只读（除非有自定义配置，否则禁止写）
  Name string `gorm:"->;<-:create"` // 允许读和写
  Name string `gorm:"->:false;<-:create"` // 仅创建（禁止从 db 读）
  Name string `gorm:"-"`  // 读写操作均会忽略该字段
}
```

## 创建 / 更新时间追踪（纳秒、毫秒、秒、Time）
GORM 约定使用 CreatedAt、UpdatedAt 追踪创建 / 更新时间。如果您定义了这种字段，GORM 在创建、更新时会自动填充 当前时间

要使用不同名称的字段，您可以配置 autoCreateTim、autoUpdateTim 标签

如果您想要保存 UNIX（毫 / 纳）秒时间戳，而不是 time，您只需简单地将 time.Time 修改为 int 即可

```go
type User struct {
  CreatedAt time.Time // 在创建时，如果该字段值为零值，则使用当前时间填充
  UpdatedAt int       // 在创建时该字段值为零值或者在更新时，使用当前时间戳秒数填充
  Updated   int64 `gorm:"autoUpdateTime:nano"` // 使用时间戳填纳秒数充更新时间
  Updated   int64 `gorm:"autoUpdateTime:milli"` // 使用时间戳毫秒数填充更新时间
  Created   int64 `gorm:"autoCreateTime"`      // 使用时间戳秒数填充创建时间
}
```



# 嵌入结构体

1. 如果匿名字段，GORM 会将其字段包含在父结构体中

   ```go
   type User struct {
     gorm.Model
     Name string
   }
   // 等效于
   type User struct {
     ID        uint           `gorm:"primaryKey"`
     CreatedAt time.Time
     UpdatedAt time.Time
     DeletedAt gorm.DeletedAt `gorm:"index"`
     Name string
   }
   ```

2. 正常的结构体，通过`embedded`嵌入，可以使用标签 `embeddedPrefix` 来为 db 中的字段名添加前缀

   ```go
   type Author struct {
       Name  string
       Email string
   }
   
   type Blog struct {
     ID      int
     Author  Author `gorm:"embedded"`
     Upvotes int32
   }
   // 等效于
   type Blog struct {
     ID    int64
       Name  string
       Email string
     Upvotes  int32
   }
   
   
   //改变前缀
   type Blog struct {
     ID      int
     Author  Author `gorm:"embedded;embeddedPrefix:author_"`
     Upvotes int32
   }
   // 等效于
   type Blog struct {
     ID          int64
       AuthorName  string
       AuthorEmail string
     Upvotes     int32
   }
   ```

   

# 字段标签表

|标签名	|说明|列子|
|-------|--------------------|-------|
|column|	指定 db 列名||
|type	|列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持 bool、int、uint、float、string、time、bytes 并且可以和其他标签一起使用，例如：not null、size, autoIncrement… 像 varbinary(8) 这样指定数据库数据类型也是支持的。在使用指定数据库数据类型时，它需要是完整的数据库数据类型，如：MEDIUMINT UNSIGNED not NULL AUTO_INSTREMENT||
|size	|指定列大小，例如：size:256||
|primaryKey	|指定列为主键||
|unique	|指定列为唯一||
|default	|指定列的默认值，插入记录到数据库时，默认值 *会被用于* 填充值为 [零值](https://tour.golang.org/basics/12) 的字段，像 `0`、`''`、`false` 等零值，不会将这些字段定义的默认值保存到数据库。您需要使用指针类型或 Scanner/Valuer 来避免这个问题 Age  *int           `gorm:"default:18"`|`Name string  `   `gorm:"default:老王"` `|
|precision	|指定列的精度||
|scale	|指定列大小||
|not null	|指定列为 NOT NULL||
|autoIncrement	|指定列为自动增长||
|embedded	|嵌套字段||
|embeddedPrefix	|嵌入字段的列名前缀||
|autoCreateTime	|创建时追踪当前时间，对于 int 字段，它会追踪时间戳秒数，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoCreateTime:nano||
|autoUpdateTime	|创建 / 更新时追踪当前时间，对于 int 字段，它会追踪时间戳秒数，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoUpdateTime:milli ||
|index	| 根据参数创建索引，多个字段使用相同的名称则创建复合索引，查看 索引 获取详情||
|uniqueIndex	|与 index 相同，但创建的是唯一索引||
|check	|创建检查约束，例如 check:age > 13，查看 约束 获取详情||
|<-	|设置字段写入的权限， <-:create 只创建、<-:update 只更新、<-:false 无写入权限、<- 创建和更新权限||
|->	|设置字段读的权限，->:false 无读权限||
|-	|忽略该字段，- 无读写权限||
| comment | 迁移时为字段添加注释 ||
