# 简介

io 是go的标准库，是对文件的操作和标准输入输出的底层

# 输入输出的底层原理

终端其实是一个文件，相关实例如下：

- `os.Stdin`：标准输入的文件实例，类型为`*File`

- `os.Stdout`：标准输出的文件实例，类型为`*File`

- `os.Stderr`：标准错误输出的文件实例，类型为`*File`

  ```go
  //以文件的方式操作终端
  var buf [16]byte
  os.Stdin.Read(buf[:])
  os.Stdin.WriteString(string(buf[:]))
  ```

**File 的内幕**

```go
//file 结构体
type File struct {
	*file // os specific
}

type file struct {
   pfd        poll.FD
   name       string
   dirinfo    *dirInfo // nil unless directory being read
   appendMode bool     // whether file is opened for appending
}
```



# 文件相关的api

根据提供的文件名创建新的文件，返回一个文件对象，默认权限是0666

 ```go
func Create(name string) (file *File, err Error)
 ```

  根据文件描述符创建相应的文件，返回一个文件对象

 ```go
func NewFile(fd uintptr, name string) *File
 ```

  只读方式打开一个名称为name的文件

 ```go
func Open(name string) (file *File, err Error)
 ```

  打开名称为name的文件，flag是打开的方式，只读、读写等，perm是权限

```go
func OpenFile(name string, flag int, perm uint32) (file *File, err Error)
```

  写入byte类型的信息到文件

```go
func (file *File) Write(b []byte) (n int, err Error)
```

  在指定位置开始写入byte类型的信息

```go
func (file *File) WriteAt(b []byte, off int64) (n int, err Error)
```

  写入string信息到文件

 ```go
func (file *File) WriteString(s string) (ret int, err Error)
 ```

  读取数据到b中

 ```go
func (file *File) Read(b []byte) (n int, err Error)
 ```

  从off开始读取数据到b中

 ```go
func (file *File) ReadAt(b []byte, off int64) (n int, err Error)
 ```

  删除文件名为name的文件

 ```go
func Remove(name string) Error
 ```

  

### bufio

- bufio包实现了带缓冲区的读写，是对文件读写的封装
- bufio缓冲写数据

| 模式        | 含义     |
| ----------- | -------- |
| os.O_WRONLY | 只写     |
| os.O_CREATE | 创建文件 |
| os.O_RDONLY | 只读     |
| os.O_RDWR   | 读写     |
| os.O_TRUNC  | 清空     |
| os.O_APPEND | 追加     |

bufio读数据
```go
import (
    "bufio"
    "fmt"
    "io"
    "os"
)

func wr() {
    // 参数2：打开模式，所有模式d都在上面
    // 参数3是权限控制
    // w写 r读 x执行   w  2   r  4   x  1
    file, err := os.OpenFile("./xxx.txt", os.O_CREATE|os.O_WRONLY, 0666)
    if err != nil {
        return
    }
    defer file.Close()
    // 获取writer对象
    writer := bufio.NewWriter(file)
    for i := 0; i < 10; i++ {
        writer.WriteString("hello\n")
    }
    // 刷新缓冲区，强制写出
    writer.Flush()
}

func re() {
    file, err := os.Open("./xxx.txt")
    if err != nil {
        return
    }
    defer file.Close()
    reader := bufio.NewReader(file)
    for {
        line, _, err := reader.ReadLine()
        if err == io.EOF {
            break
        }
        if err != nil {
            return
        }
        fmt.Println(string(line))
    }

}

func main() {
    re()
}
```



### ioutil工具包

- 工具包写文件
- 工具包读取文件

```go
import (
   "fmt"
   "io/ioutil"
)

func wr() {
   err := ioutil.WriteFile("./yyy.txt", []byte("www.5lmh.com"), 0666)
   if err != nil {
      fmt.Println(err)
      return
   }
}

func re() {
   content, err := ioutil.ReadFile("./yyy.txt")
   if err != nil {
      fmt.Println(err)
      return
   }
   fmt.Println(string(content))
}

func main() {
   re()
}
```



# 案列

`os.Open()`函数能够打开一个文件，返回一个`*File`和一个`err`。对得到的文件实例调用close()方法能够关闭文件。

```go
func main() {
    // 只读方式打开当前目录下的main.go文件
    file, err := os.Open("./main.go")
    if err != nil {
        fmt.Println("open file failed!, err:", err)
        return
    }
    // 关闭文件
    file.Close()
}
```

写文件

```go
func main() {
    // 新建文件
    file, err := os.Create("./xxx.txt")
    if err != nil {
        fmt.Println(err)
        return
    }
    defer file.Close()
    for i := 0; i < 5; i++ {
        file.WriteString("ab\n")
        file.Write([]byte("cd\n"))
    }
}
```

读文件

文件读取可以用file.Read()和file.ReadAt()，读到文件末尾会返回io.EOF的错误

```go

import (
    "fmt"
    "io"
    "os"
)
func main() {
    // 打开文件
    file, err := os.Open("./xxx.txt")
    if err != nil {
        fmt.Println("open file err :", err)
        return
    }
    defer file.Close()
    // 定义接收文件读取的字节数组
    var buf [128]byte
    var content []byte
    for {
        n, err := file.Read(buf[:])
        if err == io.EOF {
            // 读取结束
            break
        }
        if err != nil {
            fmt.Println("read file err ", err)
            return
        }
        content = append(content, buf[:n]...)
    }
    fmt.Println(string(content))
}
```

文件拷贝

```go
func main() {
    // 打开源文件
    srcFile, err := os.Open("./xxx.txt")
    if err != nil {
        fmt.Println(err)
        return
    }
    // 创建新文件
    dstFile, err2 := os.Create("./abc2.txt")
    if err2 != nil {
        fmt.Println(err2)
        return
    }
    // 缓冲读取
    buf := make([]byte, 1024)
    for {
        // 从源文件读数据
        n, err := srcFile.Read(buf)
        if err == io.EOF {
            fmt.Println("读取完毕")
            break
        }
        if err != nil {
            fmt.Println(err)
            break
        }
        //写出去
        dstFile.Write(buf[:n])
    }
    srcFile.Close()
    dstFile.Close()
}
```

