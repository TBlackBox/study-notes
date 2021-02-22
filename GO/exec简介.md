# 简介
```go
import "os/exec"
```
包exec运行外部命令。它封装了os.StartProcess，使其更容易重映射stdin和stdout，用管道连接I/O，以及做其他调整。

与 C 和其他语言的 "system "库调用不同，os/exec 包有意不调用系统 shell，也不扩展任何 glob 模式或处理其他通常由 shell 完成的扩展、管道或重定向。这个包的行为更像 C 的 "exec" 系列函数。要扩展glob模式，可以直接调用shell，注意转义任何危险的输入，或者使用path/filepath包的Glob函数。要扩展环境变量，使用包os的ExpandEnv。

注意，这个包中的例子是以Unix系统为前提的。它们可能无法在 Windows 上运行，也无法在 golang.org 和 godoc.org 使用的 Go Playground 中运行。


# 概览
```go
func LookPath(file string) (string, error)
type Cmd
    func Command(name string, arg ...string) *Cmd
    func CommandContext(ctx context.Context, name string, arg ...string) *Cmd
    func (c *Cmd) CombinedOutput() ([]byte, error)
    func (c *Cmd) Output() ([]byte, error)
    func (c *Cmd) Run() error
    func (c *Cmd) Start() error
    func (c *Cmd) StderrPipe() (io.ReadCloser, error)
    func (c *Cmd) StdinPipe() (io.WriteCloser, error)
    func (c *Cmd) StdoutPipe() (io.ReadCloser, error)
    func (c *Cmd) String() string
    func (c *Cmd) Wait() error
type Error
    func (e *Error) Error() string
    func (e *Error) Unwrap() error
type ExitError
    func (e *ExitError) Error() string
```

## 变量
```go
var ErrNotFound = errors.New("executable file not found in $PATH")
```
ErrNotFound是指在路径搜索中未能找到可执行文件时产生的错误。

## 函数
### LookPath(file string) (string, error)
```
func LookPath(file string) (string, error)
```
LookPath在PATH环境变量命名的目录中搜索一个名为文件的可执行文件。如果文件中包含斜杠，则直接尝试，不参考PATH。结果可能是绝对路径或相对于当前目录的路径。
### 例子
```go
	path, err := exec.LookPath("fortune")
	if err != nil {
		log.Fatal("installing fortune is in your future")
	}
	fmt.Printf("fortune is available at %s\n", path)
```
### Command(name string, arg ...string) *Cmd
```
func Command(name string, arg ...string) *Cmd
```
1. Command返回Cmd结构，以执行指定参数的程序。它只在返回的结构中设置Path和Args。

2. 如果name不包含路径分隔符，Command会尽可能使用LookPath将name解析为一个完整的路径，否则直接使用name作为Path。

3. 返回的Cmd的Args字段是由命令名后面的arg元素构造的，所以arg不应该包括命令名本身。例如，Command("echo"，"hello")。Args[0]总是name，而不是可能解析的Path。

4. 在 Windows 中，进程会将整个命令行作为一个单一的字符串接收，并进行自己的解析。Command 将 Args 合并并引用成一个命令行字符串，其算法与使用 CommandLineToArgvW 的应用程序兼容（这是最常见的方式）。值得注意的例外是msiexec.exe和cmd.exe（因此，所有批处理文件），它们有不同的解引号算法。在这些或其他类似的情况下，你可以自己做引号，并在SysProcAttr.CmdLine中提供完整的命令行，将Args留空。

#### 例子
```go
cmd := exec.Command("tr", "a-z", "A-Z")
cmd.Stdin = strings.NewReader("some input")
var out bytes.Buffer
cmd.Stdout = &out
err := cmd.Run()
if err != nil {
	log.Fatal(err)
}
fmt.Printf("in all caps: %q\n", out.String())
```

### CommandContext(ctx context.Context, name string, arg ...string)
```
func CommandContext(ctx context.Context, name string, arg ...string) *Cmd
```
CommandContext 与 Command 类似，但包含一个上下文。

如果上下文在命令完成之前就已经完成了，那么所提供的上下文就被用来杀死进程（通过调用os.Process.Kill）。
#### 例子
```go
ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
	defer cancel()

	if err := exec.CommandContext(ctx, "sleep", "5").Run(); err != nil {
		// This will fail after 100 milliseconds. The 5 second sleep
		// will be interrupted.
	}
```
# 类型CMD
Cmd代表一个正在准备或运行的外部命令。
Cmd在调用了它的Run、Output或CombinedOutput方法后，不能被重复使用。
```go
type Cmd struct {
	//Path是要运行的命令的路径。这是一必须设置为非零值的字段。如果Path是相对的，则相对于Dir进行评估
	Path string

	// Args存放命令行参数，包括Args[0].如果Args字段为空或为零，Run会使用{Path}。
    //在典型的使用中，Path和Args都是通过调用Command来设置的。
	Args []string
	// Env指定进程的环境。
	// 每个条目的形式是 "key=value"。
	// 如果Env为nil，则新进程使用当前进程的环境。
	// 如果Env包含重复的环境键，则只使用每个重复键的分片中的最后一个值。
	// 作为Windows上的一种特殊情况，如果SYSTEMROOT缺失，并且没有明确设置为空字符串，则总是被添加。
	Env []string

	// Dir指定命令的工作目录。
	// 如果Dir为空字符串，则Run在调用进程的当前目录下运行命令。
	Dir string

	// Stdin指定进程的标准输入。
	// 如果Stdin为nil，则进程从null设备(os.DevNull)读取。
	// 如果Stdin是一个*os.File，则进程的标准输入直接连接到该文件。
	// 否则，在命令执行过程中，会有一个单独的goroutine从Stdin中读取数据，并通过管道将这些数据传送给命令。在这种情况下，Wait不会完成，直到goroutine停止复制，原因是它已经到达了Stdin的终点（EOF或读取错误），或者是因为向管道写入数据返回了错误。
	Stdin io.Reader

	// Stdout和Stderr指定进程的标准输出和错误。
	// 如果其中之一为nil，Run会将相应的文件描述符连接到空设备(os.DevNull)。
	// 如果其中一个是*os.File，则进程的相应输出直接连接到该文件。
	// 否则，在执行命令的过程中，会有一个单独的goroutine通过管道从进程中读取数据，并将这些数据传递给相应的Writer。在这种情况下，Wait不会完成，直到goroutine达到EOF或遇到错误。
	// 如果Stdout和Stderr是同一个写程序，并且有一个可以与==进行比较的类型，那么一次最多只有一个goroutine会调用Write。
	Stdout io.Writer
	Stderr io.Writer

	// ExtraFiles指定了新进程要继承的额外打开的文件。它不包括标准输入、标准输出或标准误差。如果非nil，则条目i成为文件描述符3+i。
	// Windows上不支持ExtraFiles。
	ExtraFiles []*os.File

	// SysProcAttr 拥有可选的操作系统特定属性。
	// Run 将它作为 os.ProcAttr 的 Sys 字段传给 os.StartProcess。
	SysProcAttr *syscall.SysProcAttr

	//进程是底层进程，一旦启动。
	Process *os.Process

	// ProcessState包含关于退出的进程的信息。
	// 在调用 "等待 "或 "运行 "后可使用。
	ProcessState *os.ProcessState

	ctx             context.Context // nil表示没有
	lookPathErr     error           // LookPath错误，如果有的话。
	finished        bool            // 当等待被召唤时
	childFiles      []*os.File
	closeAfterStart []io.Closer
	closeAfterWait  []io.Closer
	goroutine       []func() error
	errch           chan error // 每条线路发送一次
	waitDone        chan struct{}
}
```



## CMD的方法
### CombinedOutput() ([]byte, error)
```
func (c *Cmd) CombinedOutput() ([]byte, error)
```
CombinedOutput运行命令并返回其组合的标准输出和标准错误。
#### 例子
```go
cmd := exec.Command("sh", "-c", "echo stdout; echo 1>&2 stderr")
	stdoutStderr, err := cmd.CombinedOutput()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", stdoutStderr)
```
### Output() ([]byte, error)
```go
func (c *Cmd) Output() ([]byte, error)
```
输出运行命令并返回其标准输出。返回的任何错误通常都是* ExitError类型。如果c.Stderr为零，则Output填充ExitError.Stderr。

###  Run() error
```go
func (c *Cmd) Run() error
```
运行将启动指定的命令并等待其完成,也就是阻塞执行。
如果命令运行，返回的错误为nil，复制stdin，stdout和stderr时没有问题，并且退出状态为零。
如果命令启动但未成功完成，则错误的类型为* ExitError。对于其他情况，可能会返回其他错误类型。
如果调用goroutine已使用runtime.LockOSThread锁定了操作系统线程并修改了任何可继承的OS级线程状态（例如Linux或Plan 9名称空间），则新进程将继承调用者的线程状态。

#### 例子
```go
cmd := exec.Command("sleep", "1")
log.Printf("Running command and waiting for it to finish...")
err := cmd.Run()
log.Printf("Command finished with error: %v", err)
```
### Start() error
```
func (c *Cmd) Start() error
```
将启动指定的命令，但不等待其完成，不阻塞执行。
如果启动成功返回，则将设置c.Process字段。
一旦命令退出，Wait方法将返回退出代码并释放关联的资源。

#### 列子
```go
cmd := exec.Command("sleep", "5")
	err := cmd.Start()
	if err != nil {
		log.Fatal(err)
	}
	log.Printf("Waiting for command to finish...")
	err = cmd.Wait()
	log.Printf("Command finished with error: %v", err)
```

### StderrPipe() (io.ReadCloser, error)
```go
func (c *Cmd) StderrPipe() (io.ReadCloser, error)
```
当命令启动时，StderrPipe返回将连接到命令的标准错误的管道。
在看到命令退出后，Wait将关闭管道，因此大多数调用者不需要自己关闭管道。因此，在管道中的所有读取完成之前调用Wait是不正确的。出于相同的原因，在使用StderrPipe时使用“Start”是不正确的。请参阅StdoutPipe示例以了解惯用用法。

#### 列子
```go
cmd := exec.Command("sh", "-c", "echo stdout; echo 1>&2 stderr")
	stderr, err := cmd.StderrPipe()
	if err != nil {
		log.Fatal(err)
	}

	if err := cmd.Start(); err != nil {
		log.Fatal(err)
	}

	slurp, _ := ioutil.ReadAll(stderr)
	fmt.Printf("%s\n", slurp)

	if err := cmd.Wait(); err != nil {
		log.Fatal(err)
	}
```
#### StdinPipe() (io.WriteCloser, error)
```go
func (c *Cmd) StdinPipe() (io.WriteCloser, error)
```
当命令启动时，StdinPipe返回将连接到命令标准输入的管道。在Wait看到命令退出后，管道将自动关闭。呼叫者只需要致电Close即可强制管道尽快关闭。例如，如果在关闭标准输入之前不会退出正在运行的命令，则调用者必须关闭管道。
#### 列子
```go
cmd := exec.Command("cat")
	stdin, err := cmd.StdinPipe()
	if err != nil {
		log.Fatal(err)
	}

	go func() {
		defer stdin.Close()
		io.WriteString(stdin, "values written to stdin are passed to cmd's standard input")
	}()

	out, err := cmd.CombinedOutput()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s\n", out)
```
### StdoutPipe() (io.ReadCloser, error)
```go
func (c *Cmd) StdoutPipe() (io.ReadCloser, error)
```
StdoutPipe返回一个管道，该管道将在命令启动时连接到命令的标准输出。
在看到命令退出后，Wait将关闭管道，因此大多数调用者不需要自己关闭管道。因此，在管道中的所有读取完成之前调用Wait是不正确的。由于相同的原因，使用StdoutPipe时调用Run是不正确的。请参阅示例以了解惯用用法。
#### 列子
```go
cmd := exec.Command("echo", "-n", `{"Name": "Bob", "Age": 32}`)
	stdout, err := cmd.StdoutPipe()
	if err != nil {
		log.Fatal(err)
	}
	if err := cmd.Start(); err != nil {
		log.Fatal(err)
	}
	var person struct {
		Name string
		Age  int
	}
	if err := json.NewDecoder(stdout).Decode(&person); err != nil {
		log.Fatal(err)
	}
	if err := cmd.Wait(); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s is %d years old\n", person.Name, person.Age)
```
### String() string
```go
func (c *Cmd) String() string
```
字符串返回人类可读的c描述。它仅用于调试。特别是，它不适合用作外壳的输入。在Go发行版中，String的输出可能会有所不同。

### Wait() error
```go
func (c *Cmd) Wait() error
```
Wait等待命令退出，并等待所有复制到stdin或从stdout或stderr复制完成。
该命令必须已经由“Start”启动。
如果命令运行，返回的错误为nil，复制stdin，stdout和stderr时没有问题，并且退出状态为零。
如果命令运行失败或未成功完成，则错误的类型为* ExitError。对于I / O问题，可能会返回其他错误类型。
如果c.Stdin，c.Stdout或c.Stderr中的任何一个都不是* os.File，则Wait也将等待相应的I / O循环复制到该进程或从该进程复制完成。
等待释放与Cmd关联的所有资源。


# 类型 Error
```go
type Error struct {
    // Name is the file name for which the error occurred.
    Name string
    // Err is the underlying error.
    Err error
}
```
## Error 的方法

###  Error() string
```go
func (e *Error) Error() string
```

### Unwrap() error
```go
func (e *Error) Unwrap() error
```

# 类型 ExitError
```go
type ExitError struct {
    *os.ProcessState

    // Stderr保存了Cmd.Output方法输出的标准错误的子集，如果没有收集标准错误的话。
    // 如果错误输出较长，Stderr可能只包含输出的前缀和后缀，中间用省略字节数的文字代替。
    // Stderr是为调试提供的，用于包含在错误信息中。
    // 有其他需求的用户应该根据需要重定向Cmd.Stderr。
    Stderr []byte // Go 1.6
}
```

## ExitError的方法
### Error() string
```go
func (e *ExitError) Error() string
```





# 案列

## 实时从管道中读取数据

```go
func RunCommand(name string, arg ...string) error {
	cmd := exec.Command(name, arg...)
    // 命令的错误输出和标准输出都连接到同一个管道
	stdout, err := cmd.StdoutPipe()
	cmd.Stderr = cmd.Stdout

	if err != nil {
		return err
	}

	if err = cmd.Start(); err != nil {
		return err
	}
    // 从管道中实时获取输出并打印到终端
	for {
		tmp := make([]byte, 1024)
		_, err := stdout.Read(tmp)
		fmt.Print(string(tmp))
		if err != nil {
			break
		}
	}

	if err = cmd.Wait(); err != nil {
		return err
	}
	return nil
}
```

# 读出数据并打印
```go
//ExecuteCommand 执行命令
func ExecuteCommand(cmdPath string, arg ...string) (string, error) {
	cmd := exec.Command(cmdPath, arg...)

	// result, err := cmd.CombinedOutput()
	// if err != nil {
	// 	fmt.Println(err)
	// 	return "", err
	// }

	// fmt.Println(string(result))

	//标准输出
	// stdout, err := cmd.StdoutPipe()
	// if err != nil {
	// 	fmt.Println("标准输出绑定错误", err)
	// 	return "", err
	// }

	//标准的错误输出
	// stderr, err := cmd.StderrPipe()
	// if err != nil {
	// 	return "", err
	// }
	// outinfo := bytes.Buffer{}
	// cmd.Stdout = &outinfo

	var out bytes.Buffer
	var stderr bytes.Buffer
	cmd.Stdout = &out
	cmd.Stderr = &stderr

	//启动命令
	// if err := cmd.Start(); err != nil {
	// 	fmt.Println("启动命令错误", err)
	// 	return "", err
	// }

	// //等待命令执行完成
	// if err := cmd.Wait(); err != nil {
	// 	fmt.Println("等待命令完成错误", err.Error())
	// 	return "", err
	// }

	err := cmd.Run()
	if err != nil {
		fmt.Println(fmt.Sprint(err) + ": " + stderr.String())
		return "", err
	}

	fmt.Println("进程PID:", cmd.ProcessState.Pid())
	fmt.Println("进程退出码：", cmd.ProcessState.Sys().(syscall.WaitStatus).ExitCode)
	fmt.Println("cmd信息:", cmd.String())

	fmt.Println("输出信息：\n", out.String())
	fmt.Println("错误信息：\n", stderr.String())

	//获取执行结果的字符串
	result := ""

	return result, nil

}
```