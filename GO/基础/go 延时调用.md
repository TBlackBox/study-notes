# 延迟调用（defer）

### 特性

1. 关键字 defer 用于注册延迟调用。
2. 这些调用直到 return 前才被执。因此，可以用来做资源清理。
3. 多个defer语句，按**先进后出**的方式执行。
4. defer语句中的变量，在defer声明时就决定了。

### 用途

1. 关闭文件句柄

2. 锁资源释放

3. 数据库连接释放



## defer 碰上闭包

```go
func main() {
    var whatever [5]struct{}
    for i := range whatever {
        defer func() { fmt.Println(i) }()
    }
}
//输出
4
4
4
4
4

//也就是说函数正常执行,由于闭包用到的变量 i 在执行的时候已经变成4,所以输出全都是4.
//要知道range 是值复制
```



## derfer f.close

```go
ype Test struct {
    name string
}

func (t *Test) Close() {
    fmt.Println(t.name, " closed")
}
func Close(t Test) {
    t.Close()
}
func main() {
    ts := []Test{{"a"}, {"b"}, {"c"}}
    for _, t := range ts {
        defer t.Close()
    }
    //输出
    c closed
    c closed
    c closed
    //这种写法注意理解 看上面的就能进行对比理解
    
    //正确的写法1
    for _, t := range ts {
        defer Close(t)
    }
    //输出 
    c closed
    b closed
    a closed
    
    //正确的写法2 不需要多定义一个函数
    for _,t :=range ts{
        t2 := t
        defer t2.Close()
    }
    //输出 
    c closed
    b closed
    a closed
}

```

结论： defer 后面的语句执行的时候，函数调用的参数会被保存下来，但是不会执行，也就是复制一份。但是并没有说struct这里的this指针如何处理，通过这个例子可以看出go语言并没有把这个明确写出来的this指针当作参数来看待。

多个 defer 注册，按 FILO 次序执行 ( 先进后出 )。哪怕函数或某个延迟调用发生错误，这些调用依旧会被执行。

1. 延迟调用参数在注册时求值或复制，可用指针或闭包 "延迟" 读取。

```go
func test() {
    x, y := 10, 20

    defer func(i int) {
        println("defer:", i, y) // y 闭包引用
    }(x) // x 被复制

    x += 10
    y += 100
    println("x =", x, "y =", y)
}

func main() {
    test()
}
//输出
x = 20 y = 120
defer: 10 120

//可以用来保存值
```

2. 滥用 defer 可能会导致性能问题，尤其是在一个 "大循环" 里。

3. #### defer 与 closure

   ```go
   func foo(a, b int) (i int, err error) {
       defer fmt.Printf("first defer err %v\n", err)
       defer func(err error) { fmt.Printf("second defer err %v\n", err) }(err) //这个函数参数会被保存
       defer func() { fmt.Printf("third defer err %v\n", err) }()
       if b == 0 {
           err = errors.New("divided by zero!")
           return
       }
   
       i = a / b
       return
   }
   
   func main() {
       foo(2, 0)
   }
   //输出
   third defer err divided by zero!
   second defer err <nil>
   first defer err <nil>
   ```

   解释：如果 defer 后面跟的不是一个 closure(关闭) 最后执行的时候我们得到的并不是最新的值。

4.defer 与 return

```go
func foo() (i int) {

    i = 0
    defer func() {
        fmt.Println(i)
    }()

    return 2
}

func main() {
    foo()
}
//输出
2
```

解释：在有具名返回值的函数中（这里具名返回值为 i），执行 return 2 的时候实际上已经将 i 的值重新赋值为 2。所以defer closure 输出结果为 2 而不是 1。



5. #### defer nil 函数

   ```go
   func test() {
       var run func() = nil
       defer run()
       fmt.Println("runs")
   }
   
   func main() {
       defer func() {
           if err := recover(); err != nil {
               fmt.Println(err)
           }
       }()
       test()
   }
   //输出
   runs
   runtime error: invalid memory address or nil pointer dereference
   ```

   解释：名为 test 的函数一直运行至结束，然后 defer 函数会被执行且会因为值为 nil 而产生 panic 异常。然而值得注意的是，run() 的声明是没有问题，因为在test函数运行完成后它才会被调用。

6. #### 在错误的位置使用 defer

   ```go
   func do() error {
       res, err := http.Get("http://www.google.com")
       defer res.Body.Close()
       if err != nil {
           return err
       }
   
       // ..code...
   
       return nil
   }
   
   func main() {
       do()
   }
   //输出
   panic: runtime error: invalid memory address or nil pointer dereference
   //因为在这里我们并没有检查我们的请求是否成功执行，当它失败的时候，我们访问了 Body 中的空变量 res ，因此会抛出异常
   if res != nil {
       defer res.Body.Close()
   }
   ```

   7 . 使用defer 需要注意，如果需要把的defer关闭的错误保存，需要定义变量存储，如果要相同的变量，释放不同的资源，则需要defer 函数定义该变量的参数

例如

```go
f, err := os.Open("book.txt")
if err != nil {
    return err
}
if f != nil {
    defer func(f io.Closer) {
        if err := f.Close(); err != nil { //定义参数
            fmt.Printf("defer close book.txt err %v\n", err)
        }
    }(f)
}

// ..code...

f, err = os.Open("another-book.txt")
if err != nil {
    return err
}
if f != nil {
    defer func(f io.Closer) { //定义参数
        if err := f.Close(); err != nil {
            fmt.Printf("defer close another-book.txt err %v\n", err)
        }
    }(f)
}
```

