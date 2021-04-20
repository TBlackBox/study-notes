

这个包可以理解为用来控goroutine的

常用的方法

```go
//让出CPU时间片，重新等待安排任务
runtime.Gosched()

//退出当前协程
runtime.Goexit()

//确定需要使用多少个OS线程来同时执行Go代码 默认值是机器上的CPU核心数。
runtime.GOMAXPROCS

//设置当前程序并发时占用的CPU逻辑核心数  //例如用核执行  即： runtime.GOMAXPROCS(1)
runtime.GOMAXPROCS()
```





