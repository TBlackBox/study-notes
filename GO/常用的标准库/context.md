# 简介

Go1.7加入了一个新的标准库context，它定义了Context类型，专门用来简化 对于处理单个请求的多个 goroutine 之间与请求域的数据、取消信号、截止时间等相关操作，这些操作可能涉及多个 API 调用。

对服务器传入的请求应该创建上下文，而对服务器的传出调用应该接受上下文。它们之间的函数调用链必须传递上下文，或者可以使用WithCancel、WithDeadline、WithTimeout或WithValue创建的派生上下文。当一个上下文被取消时，它派生的所有上下文也被取消。


>> 可以理解位go定义了一个结构体，在编程的时候，这个结构体需要在函数之间和协程之间传递，结构体里面包含了一些共同的信号，例如取消信号，截至时间等，当发出这个信号后，相应的协程进行相应的操作。



### Context接口

context.Context是一个接口，该接口定义了四个需要实现的方法。具体签名如下：

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

其中：

- Deadline方法需要返回当前Context被取消的时间，也就是完成工作的截止时间（deadline）；
- Done方法需要返回一个Channel，这个Channel会在当前工作完成或者上下文被取消之后关闭，多次调用Done方法会返回同一个Channel；
- Err方法会返回当前Context结束的原因，它只会在Done返回的Channel被关闭时才会返回非空的值；
  - 如果当前Context被取消就会返回Canceled错误；
  - 如果当前Context超时就会返回DeadlineExceeded错误；
- Value方法会从Context中返回键对应的值，对于同一个上下文来说，多次调用Value 并传入相同的Key会返回相同的结果，该方法仅用于传递跨API和进程间跟请求域的数据；



### Background()和TODO()

Go内置两个函数：**Background()**和**TODO()**，这两个函数分别**返回一个实现了Context接口的background和todo**。**我们代码中最开始都是以这两个内置的上下文对象作为最顶层的partent context，衍生出更多的子上下文对象。**

Background()主要用于main函数、初始化以及测试代码中，作为Context这个树结构的最顶层的Context，也就是根Context。

TODO()，它目前还不知道具体的使用场景，如果我们不知道该使用什么Context的时候，可以使用这个。

background和todo本质上都是emptyCtx结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context。





# With系列函数

此外，context包中还定义了四个With系列函数。

## WithCancel的函数签名

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

WithCancel返回带有新**Done通道的父节点的副本**。当调用返回的cancel函数或当关闭父上下文的Done通道时，将关闭返回上下文的Done通道，无论先发生什么情况。

取消此上下文将释放与其关联的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel。

```go
func gen(ctx context.Context) <-chan int {
        dst := make(chan int)
        n := 1
        go func() {
            for {
                select {
                case <-ctx.Done():
                    return // return结束该goroutine，防止泄露
                case dst <- n:
                    n++
                }
            }
        }()
        return dst
    }
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel() // 当我们取完需要的整数后调用cancel

    for n := range gen(ctx) {
        fmt.Println(n)
        if n == 5 {
            break
        }
    }
}

```

gen函数在单独的goroutine中生成整数并将它们发送到返回的通道。 gen的调用者在使用生成的整数之后需要取消上下文，以免gen启动的内部goroutine发生泄漏。

## WithDeadline的函数签名

```go
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
```

返回父上下文的副本，并将deadline调整为不迟于d。如果父上下文的deadline已经早于d，则WithDeadline(parent, d)在语义上等同于父上下文。当截止日过期时，当调用返回的cancel函数时，或者当父上下文的Done通道关闭时，返回上下文的Done通道将被关闭，以最先发生的情况为准。

**取消此上下文将释放与其关联的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel。**

```go
func main() {
    d := time.Now().Add(50 * time.Millisecond)
    ctx, cancel := context.WithDeadline(context.Background(), d)

    // 尽管ctx会过期，但在任何情况下调用它的cancel函数都是很好的实践。
    // 如果不这样做，可能会使上下文及其父类存活的时间超过必要的时间。
    defer cancel()

    select {
    case <-time.After(1 * time.Second):
        fmt.Println("overslept")
    case <-ctx.Done():
        fmt.Println(ctx.Err())
    }
}
```

上面的代码中，定义了一个50毫秒之后过期的deadline，然后我们调用context.WithDeadline(context.Background(), d)得到一个上下文（ctx）和一个取消函数（cancel），然后使用一个select让主程序陷入等待：等待1秒后打印overslept退出或者等待ctx过期后退出。 因为ctx50秒后就过期，所以ctx.Done()会先接收到值，上面的代码会打印ctx.Err()取消原因。



## WithTimeout的函数签名

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

WithTimeout返回WithDeadline(parent, time.Now().Add(timeout))。

取消此上下文将释放与其相关的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel，通常用于数据库或者网络连接的超时控制。

```go
var wg sync.WaitGroup

func worker(ctx context.Context) {
LOOP:
    for {
        fmt.Println("db connecting ...")
        time.Sleep(time.Millisecond * 10) // 假设正常连接数据库耗时10毫秒
        select {
        case <-ctx.Done(): // 50毫秒后自动调用
            break LOOP
        default:
        }
    }
    fmt.Println("worker done!")
    wg.Done()
}

func main() {
    // 设置一个50毫秒的超时
    ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
    wg.Add(1)
    go worker(ctx)
    time.Sleep(time.Second * 5)
    cancel() // 通知子goroutine结束
    wg.Wait()
    fmt.Println("over")
}
```



### WithValue

WithValue函数能够将请求作用域的数据与 Context 对象建立关系。声明如下：

```go
func WithValue(parent Context, key, val interface{}) Context
```

WithValue返回父节点的副本，其中与key关联的值为val。

仅对API和进程间传递请求域的数据使用上下文值，而不是使用它来传递可选参数给函数。

**所提供的键必须是可比较的，并且不应该是string类型或任何其他内置类型，以避免使用上下文在包之间发生冲突。WithValue的用户应该为键定义自己的类型。为了避免在分配给interface{}时进行分配，上下文键通常具有具体类型struct{}。或者，导出的上下文关键变量的静态类型应该是指针或接口。**

```go
package main

import (
    "context"
    "fmt"
    "sync"

    "time"
)

// context.WithValue

type TraceCode string

var wg sync.WaitGroup

func worker(ctx context.Context) {
    key := TraceCode("TRACE_CODE")
    traceCode, ok := ctx.Value(key).(string) // 在子goroutine中获取trace code
    if !ok {
        fmt.Println("invalid trace code")
    }
LOOP:
    for {
        fmt.Printf("worker, trace code:%s\n", traceCode)
        time.Sleep(time.Millisecond * 10) // 假设正常连接数据库耗时10毫秒
        select {
        case <-ctx.Done(): // 50毫秒后自动调用
            break LOOP
        default:
        }
    }
    fmt.Println("worker done!")
    wg.Done()
}

func main() {
    // 设置一个50毫秒的超时
    ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
    // 在系统的入口中设置trace code传递给后续启动的goroutine实现日志数据聚合
    ctx = context.WithValue(ctx, TraceCode("TRACE_CODE"), "12512312234")
    wg.Add(1)
    go worker(ctx)
    time.Sleep(time.Second * 5)
    cancel() // 通知子goroutine结束
    wg.Wait()
    fmt.Println("over")
}
```

## 使用Context的注意事项

- 推荐以参数的方式显示传递Context
- 以Context作为参数的函数方法，应该把Context作为第一个参数。
- 给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO()
- Context的Value相关方法应该传递请求域的必要数据，不应该用于传递可选参数
- Context是线程安全的，可以放心的在多个goroutine中传递