# 简介

Go语言内置的log包实现了简单的日志服务

log包定义了Logger类型，该类型提供了一些格式化输出的方法。本包也提供了一个预定义的“标准”logger，可以通过调用函数

- Print系列(Print|Printf|Println）     

- Fatal系列（Fatal|Fatalf|Fatalln）

- Panic系列（Panic|Panicf|Panicln）

比自行创建一个logger对象更容易使用

logger会打印每条日志信息的日期、时间，默认输出到系统的标准错误。Fatal系列函数会在写入日志信息后调用os.Exit(1)。Panic系列函数会在写入日志信息后panic。



# 日志配置

情况下的logger只会提供日志的时间信息，但我们可以改

```go
//回标准logger的输出配置
func Flags() int
//设置标准logger的输出配置
func SetFlags(flag int)

//例如
log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)

//得到日志前缀
func Prefix() string
//设置日志前缀
func SetPrefix(prefix string)

//设置日志输出位置
func SetOutput(w io.Writer) //SetOutput函数用来设置标准logger的输出目的地，默认是标准错误输出。

//例子 将日志输出到日志文件中 xx.log  一把写道init 函数中
logFile, err := os.OpenFile("./xx.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
    fmt.Println("open log file failed, err:", err)
    return
}
log.SetOutput(logFile)
log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)
log.Println("这是一条很普通的日志。")
log.SetPrefix("[小王子]")
log.Println("这是一条很普通的日志。")
```

flag 选项

```go
const (
    // 控制输出日志信息的细节，不能控制输出的顺序和格式。
    // 输出的日志在每一项后会有一个冒号分隔：例如2009/01/23 01:23:23.123123 /a/b/c/d.go:23: message
    Ldate         = 1 << iota     // 日期：2009/01/23
    Ltime                         // 时间：01:23:23
    Lmicroseconds                 // 微秒级别的时间：01:23:23.123123（用于增强Ltime位）
    Llongfile                     // 文件全路径名+行号： /a/b/c/d.go:23
    Lshortfile                    // 文件名+行号：d.go:23（会覆盖掉Llongfile）
    LUTC                          // 使用UTC时间
    LstdFlags     = Ldate | Ltime // 标准logger的初始值
)
```



# 创建logger

log标准库中还提供了一个创建新logger对象的构造函数–New

```go
func New(out io.Writer, prefix string, flag int) *Logger

//New创建一个Logger对象。其中，参数out设置日志信息写入的目的地。参数prefix会添加到生成的每一条日志前面。

//列子
logger := log.New(os.Stdout, "<New>", log.Lshortfile|log.Ldate|log.Ltime)
logger.Println("这是自定义的logger记录的日志。")
```

# 总结 

Go内置的log库功能有限，例如无法满足记录不同级别日志的情况，我们在实际的项目中根据自己的需要选择使用第三方的日志库，如logrus、zap等。