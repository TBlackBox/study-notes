# 简介

单纯地将函数并发执行是没有意义的。函数与函数间需要交换数据才能体现并发执行函数的意义。

虽然可以使用共享内存进行数据交换，但是共享内存在不同的goroutine中容易发生竞态问题。为了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必造成性能问题。

Go语言的并发模型是CSP（Communicating Sequential Processes），提倡通过通信共享内存而不是通过共享内存而实现通信。

如果说goroutine是Go程序并发的执行体，channel就是它们之间的连接。channel是可以让一个goroutine发送特定值到另一个goroutine的通信机制。

Go 语言中的通道（channel）是一种特殊的类型。通道像一个传送带或者队列，总是遵循先入先出（First In First Out）的规则，保证收发数据的顺序。每一个通道都是一个具体类型的导管，也就是声明channel的时候需要为其指定元素类型。



# channel声明和初始化

channel是一种类型，一种引用类型。声明通道类型的格式如下

```go
//声明  
var 变量 chan 元素类型

//剧烈
var ch1 chan int   // 声明一个传递整型的通道 //通道是引用类型，通道类型的空值是nil 即ch1 为nil
var ch2 chan bool  // 声明一个传递布尔型的通道
var ch3 chan []int // 声明一个传递int切片的通道
```

声明的通道后需要使用make函数初始化之后才能使用。

```go
make(chan 元素类型, [缓冲大小])
//举例    
ch4 := make(chan int)
ch5 := make(chan bool)
ch6 := make(chan []int)
```



# 操作

通道有发送（send）、接收(receive）和关闭（close）三种操作，发送和接收都使用`<-`符号。

```go
ch := make(chan int)

//发送
ch <- 10 // 把10发送到ch中

//接受
x := <- ch // 从ch中接收值并赋值给变量x
<-ch       // 从ch中接收值，忽略结果

//关闭
close(ch)
//关闭的注意事项
//关于关闭通道需要注意的事情是，只有在通知接收方goroutine所有的数据都发送完毕的时候才需要关闭通道。通道是可以被垃圾回收机制回收的，它和关闭文件是不一样的，在结束操作之后关闭文件是必须要做的，但关闭通道不是必须的。
```

### 关闭通道的特点

1. 对一个关闭的通道再发送值就会导致panic。

2. 对一个关闭的通道进行接收会一直获取值直到通道为空。

3. 对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值。

4. 关闭一个已经关闭的通道会导致panic。

   

   可以通过内置的close()函数关闭channel（如果你的管道不往里存值或者取值的时候一定记得关闭管道）



# 无缓冲区通道和有缓冲区通道

## 无缓冲区通道

无缓冲的通道又称为阻塞的通道。

```go
//初始化的时候不加缓冲区长度
ch := make(chan int)
ch <- 10  //形成死锁  发送了数据  但是没有接收
fmt.Println("发送成功") 

//编译报错
 fatal error: all goroutines are asleep - deadlock!
```

#### 解决办法

```go
func recv(c chan int) {
    ret := <-c
    fmt.Println("接收成功", ret)
}
func main() {
    ch := make(chan int)
    go recv(ch) // 启用goroutine从通道接收值
    ch <- 10
    fmt.Println("发送成功")
}
```

无缓冲通道上的发送操作会阻塞，直到另一个goroutine在该通道上执行接收操作，这时值才能发送成功，两个goroutine将继续执行。相反，如果接收操作先执行，接收方的goroutine将阻塞，直到另一个goroutine在该通道上发送一个值。

使用无缓冲通道进行通信将导致发送和接收的goroutine同步化。因此，无缓冲通道也被称为同步通道。

## 有缓冲的通道

make函数初始化通道的时候为其指定通道的容量

```go
func main() {
    ch := make(chan int, 1) // 创建一个容量为1的有缓冲区通道
    ch <- 10
    fmt.Println("发送成功")
}
```

只要通道的容量大于零，那么该通道就是有缓冲的通道，通道的容量表示通道中能存放元素的数量。容量满了就阻塞了，需要接收了后空出容量才能继续发送。

可以使用`len` 获取通道元素个数，`cap`获取通道的容量。



# 循环取值

当通过通道发送有限的数据时，我们可以通过close函数关闭通道来告知从该通道接收值的goroutine停止等待。当通道被关闭时，往该通道发送值会引发panic，从该通道里接收的值一直都是类型零值。那如何判断一个通道是否被关闭了呢？

```go
// channel 练习
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    // 开启goroutine将0~100的数发送到ch1中
    go func() {
        for i := 0; i < 100; i++ {
            ch1 <- i
        }
        close(ch1)
    }()
    // 开启goroutine从ch1中接收值，并将该值的平方发送到ch2中
    go func() {
        for {
            i, ok := <-ch1 // 通道关闭后再取值ok=false
            if !ok {
                break
            }
            ch2 <- i * i
        }
        close(ch2)
    }()
    // 在主goroutine中从ch2中接收值打印
    for i := range ch2 { // 通道关闭后会退出for range循环  推荐方式
        fmt.Println(i)
    }
}
```

两种方式在接收值的时候判断通道是否被关闭，我们通常使用的是for range的方式。



# 单向通道

有的时候我们会将通道作为参数在多个任务函数间传递，很多时候我们在不同的任务函数中使用通道都会对其进行限制，比如限制通道在函数中只能发送或只能接收。

Go语言中提供了单向通道来处理这种情况

```go
//单向通道 只接收数据
func counter(out chan<- int) {
    for i := 0; i < 100; i++ {
        out <- i
    }
    close(out)
}
//单向通道 只接收数据
func squarer(out chan<- int, in <-chan int) {
    for i := range in {
        out <- i * i
    }
    close(out)
}

//单项通道  只发送数据
func printer(in <-chan int) {
    for i := range in {
        fmt.Println(i)
    }
}

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    go counter(ch1)
    go squarer(ch2, ch1)
    printer(ch2)
}

//1.chan<- int是一个只能发送的通道，可以发送但是不能接收；
//2.<-chan int是一个只能接收的通道，可以接收但是不能发送。
```

在函数传参及任何赋值操作中将双向通道转换为单向通道是可以的，但反过来是不可以的。



#  总结

channel异常情况总结

| channel | nil  | 非空   | 空的 | 满了   | 没满   |
| ------- | ---- | ------ | ---- | ------ | ------ |
| 接收    | 阻塞 | 接受值 | 阻塞 | 接受值 | 接受值 |
| 发送    | 阻塞 | 发送值 | 发送值 | 阻塞 | 发送值 |
| 关闭    | panic | 关闭成功，读完数据后返回零值 | 关闭成功，返回零值 | 关闭成功，读完数据后返回零值 | 关闭成功，读完数据后，返回零值 |

**注意:关闭已经关闭的channel也会引发panic。**