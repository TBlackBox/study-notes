# 简介

Go语言中没有“类”的概念，也不支持“类”的继承等面向对象的概念。Go语言中通过结构体的内嵌再配合接口比面向对象具有更高的扩展性和灵活性。



## 类型定义和类型别名

1. 类型定义

```go
//将MyInt定义为int类型
type MyInt int
//自定义类型是定义了一个全新的类型。我们可以基于内置的基本类型定义，也可以通过struct定义。
//通过Type关键字的定义，MyInt就是一种新的类型，它具有int的特性。
```

2. 类型别名

   ```go
   type TypeAlias = Type
   //别名  指向的是同一个类型 只是不同的名字或者叫法，编译完成后都不会有TypeAlias该类型了
   //byte 和rune 都是类型别名
   type byte = uint8
   type rune = int32
   ```

   例子

   ```go
   //类型定义
   type NewInt int
   
   //类型别名
   type MyInt = int
   
   func main() {
       var a NewInt
       var b MyInt
   
       fmt.Printf("type of a:%T\n", a) //type of a:main.NewInt
       fmt.Printf("type of b:%T\n", b) //type of b:int
   }
   //结果显示a的类型是main.NewInt，表示main包下定义的NewInt类型。b的类型是int。
   //MyInt类型只会在代码中存在，编译完成时并不会有MyInt类型。
   ```




# 结构体

go 没得类的概念，基础信息可以表示事物的基本属性，通过结构体来表示事物的全部或部分属性。结构体的内嵌再配合接口比面向对象更具有扩展性和灵活性。



## 定义

```go
type 类型名 struct{
    字段名 字段类型
    ...
}

1.类型名：标识自定义结构体的名称，在同一个包内不能重复。
2.字段名：表示结构体字段名。结构体中的字段名必须唯一。
3.字段类型：表示结构体字段的具体类型。
4. 结构体中字段大写开头表示可公开访问，小写表示私有（仅在定义当前结构体的包中可访问）。

//举例 定义一个人的结构体
type person struct {
        name string
        city string
        age  int8
    	phone, weight int   //同一个类型可以定义在一行
}
```



## 实列化

只有结构体实列化了，才能真正分配内存地址，也才能使用，结构体本身是一种类型，可以像内置类型一样使用。

```go
var 结构体实例 结构体类型
```

例子

```go
var p persion  //这样都代表初始化了，厘面的属性都有对应的默认值
p.name = "老王"
p.city = "重庆"
...
```



### 匿名结构体

```go
var user struct{Name string; Age int}
user.Name = "pprof.cn"
user.Age = 18

//可以用于临时场景
```



### 指针类型结构体

```go
//通过 new() 对结构体进行实列化，得到结构体的地址
var p = new(persion)

//这样p 都是一个 persion 类型的指针
//可以对结构体这样赋值
p.name = "老李"
p.age = 15
//特别注意: Go语言中支持对结构体指针直接使用.来访问结构体的成员。
```



### 取结构体的地址实列化

```go
p3 := &person{}
fmt.Printf("%T\n", p3)     //*main.person
fmt.Printf("p3=%#v\n", p3) //p3=&main.person{name:"", city:"", age:0}
p3.name = "博客"
p3.age = 30
p3.city = "成都"
//p3.name = "博客"其实在底层是(*p3).name = "博客"，这是Go语言帮我们实现的语法糖。
```



### 使用键值对初始化

```go
//建对应结构体的字段，值对应字段的初始值
p5 := person{
    name: "pprof.cn",
    city: "北京",
    age:  18,
}

//对结构体指针进行键值对初始化
p6 := &person{
    name: "pprof.cn",
    city: "北京",
    //age:  18,  没有初始值的时候可以不写 初始值为该字段的零值
}
```



### 使用值的列表初始化

```go
//简写  只写值  不写键
p8 := &person{
    "pprof.cn",
    "北京",
    18,
}

//1.必须初始化结构体的所有字段。
//2.初始值的填充顺序必须与字段在结构体中的声明顺序一致。
//3.该方式不能和键值初始化方式混用。
```



## 构造函数

go 的结构体没得构造函数，可以自己实现

```go
func newPerson(name, city string, age int8) *person {
    return &person{
        name: name,
        city: city,
        age:  age,
    }
}
//返回指针类型 值拷贝开销比较大

//调用
p9 := newPerson("二狗", "测试", 90)
```





## 方法和接收者

Go语言中的方法（Method）是一种作用于特定类型变量的函数。这种特定类型变量叫做接收者（Receiver）。接收者的概念就类似于其他语言中的this或者 self。

方法定义的格式

```go
 func (接收者变量 接收者类型) 方法名(参数列表) (返回参数) {
        函数体
    }
//1.接收者变量：接收者中的参数变量名在命名时，官方建议使用接收者类型名的第一个小写字母，而不是self、this之类的命名。例如，Person类型的接收者变量应该命名为 p，Connector类型的接收者变量应该命名为c等。
//2.接收者类型：接收者类型和参数类似，可以是指针类型和非指针类型。
//3.方法名、参数列表、返回参数：具体格式与函数定义相同。

//例如
//Person 结构体
type Person struct {
    name string
    age  int8
}

//NewPerson 构造函数
func NewPerson(name string, age int8) *Person {
    return &Person{
        name: name,
        age:  age,
    }
}

//Dream Person做梦的方法
//
func (p Person) Dream() {
    fmt.Printf("%s的梦想是学好Go语言！\n", p.name)
}

func main() {
    p1 := NewPerson("测试", 25)
    p1.Dream()
}
```

方法与函数的区别是：

- 函数不属于任何类型
- 方法属于特定的类型。



### 指针类型的接收者和值类型的接收者

接收者是一个结构体的指针，调用方法时，修改任意变量，修改都是有效的，相当于其他语言的this和self。

值类型的接收者 Go语言会在代码运行时将接收者的值复制一份。在值类型接收者的方法中可以获取接收者的成员值，但修改操作只是针对副本，无法修改接收者变量本身。

```go
// SetAge 设置p的年龄
// 使用指针接收者
func (p *Person) SetAge(newAge int8) {
    p.age = newAge
}
//使用值类型的接收者
func (p Person) SetAge1(newAge int8) {
    p.age = newAge
}

func main() {
    p1 := NewPerson("测试", 25)
    fmt.Println(p1.age) // 25
    
    p1.SetAge1(30);
    fmt.Println(p1.age) // 25
    p1.SetAge(30)
    fmt.Println(p1.age) // 30
}

//1.需要修改接收者中的值
//2.接收者是拷贝代价比较大的大对象
//3.保证一致性，如果有某个方法使用了指针接收者，那么其他的方法也应该使用指针接收者。
```



### 任意类型都能添加方法

```go
//MyInt 将int定义为自定义MyInt类型
type MyInt int

//SayHello 为MyInt添加一个SayHello的方法
func (m MyInt) SayHello() {
    fmt.Println("Hello, 我是一个int。")
}
```

注意事项： 非本地类型不能定义方法，也就是说我们不能给别的包的类型定义方法。



## 结构体的匿名字段

声明时只有类型，没有名字，没得名字的字段就是匿名字段

```go
//Person 结构体Person类型
type Person struct {
    string
    int
}

//赋值方式
p1 := Person{
        "pprof.cn",
        18,
    }
//要求
//1. 匿名字段默认是用类型名作为字段名，结构体字段名必须唯一，因此一个结构体中同种类型的匿名字段只能有一个。
```



## 嵌套结构体和匿名结构体

一个结构体中可以嵌套包含另一个结构体或结构体指针。没得字段名的，就是匿名结构体。

```go
//Address 地址结构体
type Address struct {
    Province string
    City     string
}

//User 用户结构体
type User struct {
    Name    string
    Gender  string
    address Address  //这个是嵌套不匿名结构体
    Address //匿名结构体
}

//使用方式
func main() {
    user1 := User{
        Name:   "pprof",
        Gender: "女",
        Address: Address{
            Province: "黑龙江",
            City:     "哈尔滨",
        },
    }
    
    //嵌套匿名结构体.字段名 的方式赋值
    user1.Address.Province = "重庆"
    
    //直接访问匿名结构体的字段名
    user1.Province = "成都"
    
    fmt.Printf("user1=%#v\n", user1)
    //user1=main.User{Name:"pprof", Gender:"女", Address:main.Address{Province:"黑龙江", City:"哈尔滨"}}
}
//注意
//当访问结构体成员时会先在结构体中查找该字段，找不到再去匿名结构体中查找
//如果不同匿名结构体存在相同的字段名  这个时候都需要指定具体结构体的字段了
```



## 结构体的继承

Go语言中使用结构体也可以实现其他编程语言中面向对象的继承。

```go
//Animal 动物
type Animal struct {
    name string
}

func (a *Animal) move() {
    fmt.Printf("%s会动！\n", a.name)
}

//Dog 狗
type Dog struct {
    Feet    int8
    *Animal //通过嵌套匿名结构体实现继承
}

func (d *Dog) wang() {
    fmt.Printf("%s会汪汪汪~\n", d.name)
}

func main() {
    d1 := &Dog{
        Feet: 4,
        Animal: &Animal{ //注意嵌套的是结构体指针
            name: "乐乐",
        },
    }
    d1.wang() //乐乐会汪汪汪~
    d1.move() //乐乐会动！
}
```



#  结构体与JSON序列化

- JSON 序列化函数 `data, err := json.Marshal(c)`
- 反序列化函数`err = json.Unmarshal([]byte(str), c1)`

```go
//Student 学生
type Student struct {
    ID     int
    Gender string
    Name   string
}

//Class 班级
type Class struct {
    Title    string
    Students []*Student
}

func main() {
    c := &Class{
        Title:    "101",
        Students: make([]*Student, 0, 200),
    }
    for i := 0; i < 10; i++ {
        stu := &Student{
            Name:   fmt.Sprintf("stu%02d", i),
            Gender: "男",
            ID:     i,
        }
        c.Students = append(c.Students, stu)
    }
    //JSON序列化：结构体-->JSON格式的字符串
    data, err := json.Marshal(c)
    if err != nil {
        fmt.Println("json marshal failed")
        return
    }
    fmt.Printf("json:%s\n", data)
    //JSON反序列化：JSON格式的字符串-->结构体
    str := `{"Title":"101","Students":[{"ID":0,"Gender":"男","Name":"stu00"},{"ID":1,"Gender":"男","Name":"stu01"},{"ID":2,"Gender":"男","Name":"stu02"},{"ID":3,"Gender":"男","Name":"stu03"},{"ID":4,"Gender":"男","Name":"stu04"},{"ID":5,"Gender":"男","Name":"stu05"},{"ID":6,"Gender":"男","Name":"stu06"},{"ID":7,"Gender":"男","Name":"stu07"},{"ID":8,"Gender":"男","Name":"stu08"},{"ID":9,"Gender":"男","Name":"stu09"}]}`
    c1 := &Class{}
    err = json.Unmarshal([]byte(str), c1)
    if err != nil {
        fmt.Println("json unmarshal failed!")
        return
    }
    fmt.Printf("%#v\n", c1)
}
```



# 结构体标签

Tag是结构体的元信息，可以在运行的时候通过反射的机制读取出来。（可以理解为java的注解）

Tag在结构体字段的后方定义，由一对反引号包裹起来，具体的格式如下：

```go
`key1:"value1" key2:"value2"`

//格式说明
1. 一个或多个建值组成，键与值使用冒号分隔，值用双引号括起来
2. 键值对之间使用一个空格分隔。
3. 标签也需要代码的支持
```

注意事项： 为结构体编写Tag时，必须严格遵守键值对的规则。结构体标签的解析代码的容错能力很差，一旦格式写错，编译和运行时都不会提示任何错误，通过反射也无法正确取值。例如不要在key和value之间添加空格。

```go
//Student 学生
type Student struct {
    ID     int    `json:"id"` //通过指定tag实现json序列化该字段时的key
    Gender string //json序列化是默认使用字段名作为key
    name   string //私有不能被json包访问
}

func main() {
    s1 := Student{
        ID:     1,
        Gender: "女",
        name:   "pprof",
    }
    data, err := json.Marshal(s1)
    if err != nil {
        fmt.Println("json marshal failed!")
        return
    }
    fmt.Printf("json str:%s\n", data) //json str:{"id":1,"Gender":"女"}
}
```

