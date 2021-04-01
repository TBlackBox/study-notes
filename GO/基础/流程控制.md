## if

```go
• 可省略条件表达式括号。
• 持初始化语句，可定义代码块局部变量。 
• 代码块左 括号必须在条件表达式尾部。
. 不支持三元操作符(三目运算符) "a > b ? a : b"

if 布尔表达式 {
    /* 在布尔表达式为 true 时执行 */
}

//列子 例子包含 if  和if{}else if{}  和 if{}else{}的用法

 x := 0

// if x > 10        // Error: missing condition in if statement
// {
// }

if n := "abc"; x > 0 {     // 初始化语句未必就是定义变量， 如 println("init") 也是可以的。
    println(n[2])
} else if x < 0 {    // 注意 else if 和 else 左大括号位置。
    println(n[1])
} else {
    println(n[0])
}

```



## switch

- switch 语句用于基于不同条件执行不同动作，每一个 case 分支都是唯一的，从上直下逐一测试，直到匹配为止。

- Golang switch 分支表达式可以是任意类型，不限于常量。可省略 break，默认自动终止。(这点跟其他语言有点不同)
- 可用于多个可能符合的条件使用`,`分割，例如：case val1, val2, val3

```go
//语法
switch var1 {
    case val1:
        ...
    case val2，val3: //多个条件
        ...
    default:
        ...
}

//列子

import "fmt"

func main() {
   /* 定义局部变量 */
   var grade string = "B"
   var marks int = 90

   switch marks {
      case 90: grade = "A"
      case 80: grade = "B"
      case 50,60,70 : grade = "C"
      default: grade = "D"  
   }

   switch {
      case grade == "A" :
         fmt.Printf("优秀!\n" )     
      case grade == "B", grade == "C" :
         fmt.Printf("良好\n" )      
      case grade == "D" :
         fmt.Printf("及格\n" )      
      case grade == "F":
         fmt.Printf("不及格\n" )
      default:
         fmt.Printf("差\n" )
   }
   fmt.Printf("你的等级是 %s\n", grade )
}
```



# Type Switch

switch 语句还可以被用于 type-switch 来判断某个 interface 变量中实际存储的变量类型。

```go
//语法
switch x.(type){
    case type:
       statement(s)      
    case type:
       statement(s)
    /* 你可以定义任意个数的case */
    default: /* 可选 */
       statement(s)
}

//例子
func main() {
    var x interface{}
    //写法一：
    switch i := x.(type) { // 带初始化语句
    case nil:
        fmt.Printf(" x 的类型 :%T\r\n", i)
    case int:
        fmt.Printf("x 是 int 型")
    case float64:
        fmt.Printf("x 是 float64 型")
    case func(int) float64:
        fmt.Printf("x 是 func(int) 型")
    case bool, string:
        fmt.Printf("x 是 bool 或 string 型")
    default:
        fmt.Printf("未知型")
    }
    //写法二
    var j = 0
    switch j {
    case 0:
    case 1:
        fmt.Println("1")
    case 2:
        fmt.Println("2")
    default:
        fmt.Println("def")
    }
    //写法三
    var k = 0
    switch k {
    case 0:
        println("fallthrough")
        fallthrough
        /*
            Go的switch非常灵活，表达式不必是常量或整数，执行的过程从上至下，直到找到匹配项；
            而如果switch没有表达式，它会匹配true。
            Go里面switch默认相当于每个case最后带有break，
            匹配成功后不会自动向下执行其他case，而是跳出整个switch,
            但是可以使用fallthrough强制执行后面的case代码。
        */
    case 1:
        fmt.Println("1")
    case 2:
        fmt.Println("2")
    default:
        fmt.Println("def")
    }
    //写法三
    var m = 0
    switch m {
    case 0, 1:
        fmt.Println("1")
    case 2:
        fmt.Println("2")
    default:
        fmt.Println("def")
    }
    //写法四
    var n = 0
    switch { //省略条件表达式，可当 if...else if...else
    case n > 0 && n < 10:
        fmt.Println("i > 0 and i < 10")
    case n > 10 && n < 20:
        fmt.Println("i > 10 and i < 20")
    default:
        fmt.Println("def")
    }
}

//输出结果
x 的类型 :<nil>
fallthrough
1
1
def
```



# 条件语句select

- select 语句类似于 switch 语句，但是select会随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。

- select 是Go中的一个控制结构，类似于用于通信的switch语句。每个case必须是一个通信操作，要么是发送要么是接收。

-  select 随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。一个默认的子句应该总是可运行的。

```go
//语法

```

```go
select {
    case communication clause  :  //communication 通信，传播 clause 语句
       statement(s);      
    case communication clause  :
       statement(s);
    /* 你可以定义任意数量的 case */
    default : /* 可选 */
       statement(s); //statement 声明，陈述
}

每个case都必须是一个通信
所有channel表达式都会被求值
所有被发送的表达式都会被求值
如果任意某个通信可以进行，它就执行；其他被忽略。
如果有多个case都可以运行，Select会随机公平地选出一个执行。其他不会执行。
否则：
如果有default子句，则执行该语句。
如果没有default字句，select将阻塞，直到某个通信可以运行；Go不会重新对channel或值进行求值。

//例子
func main() {
   var c1, c2, c3 chan int
   var i1, i2 int
   select {
      case i1 = <-c1:
         fmt.Printf("received ", i1, " from c1\n")
      case c2 <- i2:
         fmt.Printf("sent ", i2, " to c2\n")
      case i3, ok := (<-c3):  // same as: i3, ok := <-c3
         if ok {
            fmt.Printf("received ", i3, " from c3\n")
         } else {
            fmt.Printf("c3 is closed\n")
         }
      default:
         fmt.Printf("no communication\n")
   }    
}
//输出
//no communication
```

流程描述

```go
select { //不停的在这里检测
    case <-chanl : //检测有没有数据可以读
    //如果chanl成功读取到数据，则进行该case处理语句
    case chan2 <- 1 : //检测有没有可以写
    //如果成功向chan2写入数据，则进行该case处理语句


    //假如没有default，那么在以上两个条件都不成立的情况下，就会在此阻塞//一般default会不写在里面，select中的default子句总是可运行的，因为会很消耗CPU资源
    default:
    //如果以上都没有符合条件，那么则进行default处理流程
    }
```

在一个select语句中，Go会按顺序从头到尾评估每一个发送和接收的语句。

如果其中的任意一个语句可以继续执行（即没有被阻塞），那么就从那些可以执行的语句中任意选择一条来使用。 如果没有任意一条语句可以执行（即所有的通道都被阻塞），那么有两种可能的情况： 

- ①如果给出了default语句，那么就会执行default的流程，同时程序的执行会从select语句后的语句中恢复。 
- ②如果没有default语句，那么select语句将被阻塞，直到至少有一个case可以进行下去。

### 典型用法

```go
//比如在下面的场景中，使用全局resChan来接受response，如果时间超过3S,resChan中还没有数据返回，则第二条case将执行
var resChan = make(chan int)
// do request
func test() {
    select {
    case data := <-resChan:
        doData(data)
    case <-time.After(time.Second * 3):
        fmt.Println("request time out")
    }
}

func doData(data int) {
    //...
}



//退出
//主线程（协程）中如下：
var shouldQuit=make(chan struct{})
fun main(){
    {
        //loop
    }
    //...out of the loop
    select {
        case <-c.shouldQuit:
            cleanUp()
            return
        default:
        }
    //...
}

//再另外一个协程中，如果运行遇到非法操作或不可处理的错误，就向shouldQuit发送数据通知程序停止运行
close(shouldQuit)




//在某些情况下是存在不希望channel缓存满了的需求的，可以用如下方法判断
ch := make (chan int, 5)
//...
data：=0
select {
case ch <- data:
default:
    //做相应操作，比如丢弃data。视需求而定
}
```

总结：

1. select 类似switch,最大区别是 case 是通道
2. 没有可运行的case 如果有默认，则执行默认，如果没得，者堵塞，如果有case ,者随机执行一个可用的。



# for

for循环是一个循环控制结构，可以执行指定次数的循环。

3种形式

```go
for init; condition; post { }
for condition { }
for { }

init： 一般为赋值表达式，给控制变量赋初值；
condition： 关系表达式或逻辑表达式，循环控制条件；
post： 一般为赋值表达式，给控制变量增量或减量。
for语句执行过程如下：
①先对表达式 init 赋初值；
②判别赋值表达式 init 是否满足给定 condition 条件，若其值为真，满足循环条件，则执行循环体内语句，然后执行 post，进入第二次循环，再判别 condition；否则判断 condition 的值为假，不满足条件，就终止for循环，执行循环体外语句。

//列子
s := "abc"

for i, n := 0, len(s); i < n; i++ { // 常见的 for 循环，支持初始化语句。
    println(s[i])
}

n := len(s)
for n > 0 {                // 替代 while (n > 0) {}
    println(s[n])        // 替代 for (; n > 0;) {}
    n-- 
}

for {                    // 替代 while (true) {}
    println(s)            // 替代 for (;;) {}
}


//循环嵌套
for [condition |  ( init; condition; increment ) | Range]
{
   for [condition |  ( init; condition; increment ) | Range]
   {
      statement(s)
   }
   statement(s)
}

//无线循环
for true  {
    fmt.Printf("这是无限循环。\n");
}

```



# range

range类似迭代器操作，返回 (索引, 值) 或 (键, 值)。

```go
//for 循环的 range 格式可以对 slice、map、数组、字符串等进行迭代循环。
//可忽略不想要的返回值，或 "_" 这个特殊变量。
for key, value := range oldMap {
    newMap[key] = value
}

//列子
s := "abc"
// 忽略 2nd value，支持 string/array/slice/map。
for i := range s {
    println(s[i])
}
// 忽略 index。
for _, c := range s {
    println(c)
}
// 忽略全部返回值，仅迭代。
for range s {

}

m := map[string]int{"a": 1, "b": 2}
// 返回 (key, value)。
for k, v := range m {
    println(k, v)
}


//注意
1. range 会复制对象，如果修改的化，建议使用引用类型，底层数据不会被复制
```

|             | 1st value | 2nd value |               |
| ----------- | --------- | --------- | ------------- |
| string      | index     | s[index]  | unicode, rune |
| array/slice | index     | s[index]  |               |
| map         | key       | m[key]    |               |
| channel     | element   |           |               |

另外两种引用类型 map、channel 是指针包装，而不像 slice 是 struct。

### for 和 for range区别

主要是使用场景不同

for可以

- 遍历array和slice

- 遍历key为整型递增的map

- 遍历string

for range可以完成所有for可以做的事情，却能做到for不能做的，包括

- 遍历key为string类型的map并同时获取key和value

- 遍历channel



#  循环控制Goto、Break、Continue

```go
1.三个语句都可以配合标签(label)使用
2.标签名区分大小写，定以后若不使用会造成编译错误
3.continue、break配合标签(label)可用于多层循环跳出
4.goto是调整执行位置，与continue、break配合标签(label)的结果并不相同
```

