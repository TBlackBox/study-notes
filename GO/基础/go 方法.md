#  方法的定义

Golang 方法总是绑定对象实例，并隐式将实例作为第一实参 (receiver)。

• 只能为当前包内命名类型定义方法。
• 参数 receiver 可任意命名。如方法中未曾使用 ，可省略参数名。
• 参数 receiver 类型可以是 T 或 *T。基类型 T 不能是接口或指针。 
• 不支持方法重载，receiver 只是参数签名的组成部分。
• 可用实例 value 或 pointer 调用全部方法，编译器自动转换。

```go 
func (recevier type) methodName(参数列表)(返回值列表){}

参数和返回值可以省略


//列子
type Test struct{}

// 无参数、无返回值
func (t Test) method0() {

}

// 单参数、无返回值
func (t Test) method1(i int) {

}

// 多参数、无返回值
func (t Test) method2(x, y int) {

}

// 无参数、单返回值
func (t Test) method3() (i int) {
    return
}

// 多参数、多返回值
func (t Test) method4(x, y int) (z int, err error) {
    return
}

// 无参数、无返回值
func (t *Test) method5() {

}

// 单参数、无返回值
func (t *Test) method6(i int) {

}

// 多参数、无返回值
func (t *Test) method7(x, y int) {

}

// 无参数、单返回值
func (t *Test) method8() (i int) {
    return
}

// 多参数、多返回值
func (t *Test) method9(x, y int) (z int, err error) {
    return
}
```

**注意：**

方法的接收者如果是指针类型，不管是值调用，还在指针调用，都是修改原结构体。

同理，如果是接收者是值类型，那么不管是指针调用还是值调用，都会复制一份副本。



## 普通函数和方法的区别

1. 对于普通函数，接收者为值类型时，不能将指针类型的数据直接传递，反之亦然。

```go
//1.普通函数   //调用时只能传值  即： a :=5   valueIntTest(a)
//接收值类型参数的函数
func valueIntTest(a int) int {
    return a + 10
}

//接收指针类型参数的函数 //调用时只能指针 即： a :=5   pointerIntTest(&a)
func pointerIntTest(a *int) int {
    return *a + 10
}
```

2. 对于方法（如struct的方法），接收者为值类型时，可以直接用指针类型的变量调用方法，反过来同样也可以。

   ```go
   //结构体
   type PersonD struct {
       id   int
       name string
   }
   
   //接收者为值类型  //调用既可以用指针类型调用  也可以值类型调用  
   func (p PersonD) valueShowName() {
       fmt.Println(p.name)
   }
   
   //接收者为指针类型
   func (p *PersonD) pointShowName() {
       fmt.Println(p.name)
   }
   
   //值调用方式
   personValue := PersonD{101, "hello world"}    
   personValue.valueShowName()  
   personValue.pointShowName()
   
   //指针调用方式
   personPointer := &PersonD{101, "hello world"} 
   personPointer.valueShowName()
   personPointer.pointShowName()
   
   ```

   

# 匿名字段

Golang匿名字段 ：可以像字段成员那样访问匿名字段方法，编译器负责查找。

```go
type User struct {
    id   int
    name string
}

type Manager struct {
    User
}

func (self *User) ToString() string { // receiver = &(Manager.User)
    return fmt.Sprintf("User: %p, %v", self, self)
}

func main() {
    m := Manager{User{1, "Tom"}}
    fmt.Printf("Manager: %p\n", &m)
    fmt.Println(m.ToString())
}
//输出
Manager: 0xc42000a060
User: 0xc42000a060, &{1 Tom}
```

通过匿名字段，可获得和继承类似的复用能力。依据编译器查找次序，只需在外层定义同名方法，就可以实现 "override"。

```go
type User struct {
    id   int
    name string
}

type Manager struct {
    User
    title string
}

func (self *User) ToString() string {
    return fmt.Sprintf("User: %p, %v", self, self)
}

func (self *Manager) ToString() string {
    return fmt.Sprintf("Manager: %p, %v", self, self)
}

func main() {
    m := Manager{User{1, "Tom"}, "Administrator"}

    fmt.Println(m.ToString())

    fmt.Println(m.User.ToString())
}

//输出
Manager: 0xc420074180, &{{1 Tom} Administrator}
User: 0xc420074180, &{1 Tom}
```



# 方法集

Golang方法集 ：每个类型都有与之关联的方法集，这会影响到接口实现规则

 • 类型 T 方法集包含全部 receiver T 方法。
 • 类型 `*T` 方法集包含全部 receiver T + `*T`方法。
 • 如类型 S 包含匿名字段 T，则 S 和 `*S` 方法集包含 T 方法。(也就是不能调用 T的指针类型方法) 
 • 如类型 S 包含匿名字段 `*T`，则 S 和 `*S` 方法集包含 T + *T 方法。 
 • 不管嵌入 T 或 `*T`，`*S` 方法集总是包含 T + `*T` 方法。

用实例 value 和 pointer 调用方法 (含匿名字段) 不受方法集约束，编译器总是查找全部方法，并自动转换 receiver 实参。



# 表达式

Golang 表达式 ：根据调用者不同，方法分为两种表现形式:

```go
instance.method(args...) ---> <type>.func(instance, args...)
//前者称为 method value，后者 method expression。
```

两者都可像普通函数那样赋值和传参，区别在于 method value 绑定实例，而 method expression 则须显式传参。

```go
type User struct {
    id   int
    name string
}

func (self *User) Test() {
    fmt.Printf("%p, %v\n", self, self)
}

func main() {
    u := User{1, "Tom"}
    u.Test()

    mValue := u.Test //method value 会复制 receiver。 立即复制 receiver，因为不是指针类型，不受后续修改影响。method value 和闭包的实现方式相同，实际返回 FuncVal 类型对象
    mValue() // 隐式传递 receiver

    mExpression := (*User).Test
    mExpression(&u) // 显式传递 receiver
}
//输出
0xc42000a060, &{1 Tom}
0xc42000a060, &{1 Tom}
0xc42000a060, &{1 Tom}
```

