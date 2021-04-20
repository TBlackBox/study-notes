# 安装

```go
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite
```



# 基本使用

```go
import (
	"fmt"
	"log"

	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
)

type Product struct {
	gorm.Model
	Code  string
	Price uint
}

//GromTest 测试
func GromTest() {
	// github.com/mattn/go-sqlite3
	var dbFile = "gorm.db"
	dialector := sqlite.Open(dbFile)
	db, err := gorm.Open(dialector, &gorm.Config{})
	if err != nil {
		log.Fatalln("数据录配置打开错误,err:" + err.Error())
	}

	// 迁移 schema
	db.AutoMigrate(&Product{})

	//插入数据
	product := &Product{Code: "123456",Price: 20}
	db.Create(product)

	//查询
	db.First(&product,1) //根据主键查找
	fmt.Printf("通过主键查询的数据：%v,%d",product.Code,product.Price) //通过主键查询的数据：123456,20

	//通过code 查询
	db.First(&product,"Code=?","123456")
	fmt.Println()
	fmt.Printf("通过code查询的数据：%v,%d",product.Code,product.Price)//过code查询的数据：123456,20

	//更新 全部更新
	db.Model(&product).Update("Price",200)


	db.First(&product,1) //根据主键查找
	fmt.Printf("查看数据：%s,%d",product.Code,product.Price) //查看数据：123456,200

	//多字段更新
	db.Model(&product).Updates(Product{Price: 300, Code: "654321"}) //仅更新非零值字段
	db.Model(&product).Updates(map[string]interface{}{"Price": 400,"Code": "789"})

	fmt.Println()
	fmt.Printf("查看数据：%s,%d",product.Code,product.Price)//查看数据：789,400

	//删除
	db.Delete(&product,1)
}
```

