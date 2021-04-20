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



File 的内幕

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

  