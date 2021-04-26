# 数据格式

- 是系统中数据交互不可缺少的内容

- 这里主要介绍JSON、XML、MSGPack



# JSON

- json是完全独立于语言的文本格式，是k-v的形式 name:zs
- 应用场景：前后端交互，系统间数据交互

- json使用go语言内置的encoding/json 标准库
- 编码json使用json.Marshal()函数可以对一组数据进行JSON格式的编码

```go
func Marshal(v interface{}) ([]byte, error)
```

示例过结构体生成json

```go

import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Name  string
    Hobby string
}

func main() {
    p := Person{"5lmh.com", "女"}
    // 编码json
    b, err := json.Marshal(p)
    if err != nil {
        fmt.Println("json err ", err)
    }
    fmt.Println(string(b))

    // 格式化输出
    b, err = json.MarshalIndent(p, "", "     ")
    if err != nil {
        fmt.Println("json err ", err)
    }
    fmt.Println(string(b))
}
```

##  struct tag

### Tag的含义

- `omitempty`：空值、`nil`、数组或集合长度为`0`时，忽略该字段。
- `-`：破折号，忽略该字段。
- `-,`：将字段名改为`-`。
- `,string`：将`int`类型字段转为`string`类型。

### 案列

定义结构体，其中`R1`为常规`tag`。

`R2`中：（注意返回字段的大小写）

- `code,string`：将`int`类型的`Code`转为`string`类型的`code`；
- `,string`：将`Code2`转为`string`的`Code2`；
- `msg,omitempty`：当`Msg`为`nil`或`""`时，忽略该字段，反之返回`msg`字段；
- `,omitempty`: 当`Data`为`nil`等情况时，忽略该字段，反之返回`Data`字段；
- `-`：忽略该字段；
- `-,`：将字段名改为`-`；

```go
type (
    R1 struct {
        Code int `json:"code"`
        Msg string `json:"msg"`
        Data interface{} `json:"data"`
    }

    R2 struct {
        Code int `json:"code,string"`
        Code2 int `json:",string"`
        Msg string `json:"msg,omitempty"`
        Data interface{} `json:",omitempty"`
        Data2 interface{} `json:",omitempty"`
        Id int `json:"-"`
        Id2 int `json:"-,"`
    }
)

func main() {
    r1, _ := json.Marshal(R1{Code: 0, Msg: "请求成功", Data: "字符串数据"})
    fmt.Println(string(r1)) // {"code":0,"msg":"请求成功","data":"字符串数据"}

    r2, _ := json.Marshal(R2{Code: 0, Msg: "", Data: "字符串数据", Data2: nil, Code2: 999, Id: 1000, Id2:2000})
    fmt.Println(string(r2)) // {"code":"0","Code2":"999","Data":"字符串数据","-":2000}
}
```
```go
type Person struct {
    //"-"是忽略的意思
    Name  string `json:"-"`
    Hobby string `json:"hobby" `
}
```

示例通过map生成json

```go
func main() {
    student := make(map[string]interface{})
    student["name"] = "5lmh.com"
    student["age"] = 18
    student["sex"] = "man"
    b, err := json.Marshal(student)
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(b)
}
```

- 解码json使用json.Unmarshal()函数可以对一组数据进行JSON格式的解码

```go
func Unmarshal(data []byte, v interface{}) error
```

示例解析到结构体

```go
type Person struct {
    Age       int    `json:"age,string"`
    Name      string `json:"name"`
    Niubility bool   `json:"niubility"`
}

func main() {
    // 假数据
    b := []byte(`{"age":"18","name":"5lmh.com","marry":false}`)
    var p Person
    err := json.Unmarshal(b, &p)
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(p)
}
```

示例解析到interface

```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    // 假数据
    // int和float64都当float64
    b := []byte(`{"age":1.3,"name":"5lmh.com","marry":false}`)

    // 声明接口
    var i interface{}
    err := json.Unmarshal(b, &i)
    if err != nil {
        fmt.Println(err)
    }
    // 自动转到map
    fmt.Println(i)
    // 可以判断类型
    m := i.(map[string]interface{})
    for k, v := range m {
        switch vv := v.(type) {
        case float64:
            fmt.Println(k, "是float64类型", vv)
        case string:
            fmt.Println(k, "是string类型", vv)
        default:
            fmt.Println("其他")
        }
    }
}
```



# XML

- 是可扩展标记语言，包含声明、根标签、子元素和属性
- 应用场景：配置文件以及webService

示例：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<servers version="1">
    <server>
        <serverName>Shanghai_VPN</serverName>
        <serverIP>127.0.0.1</serverIP>
    </server>
    <server>
        <serverName>Beijing_VPN</serverName>
        <serverIP>127.0.0.2</serverIP>
    </server>
</servers>
```



```go

package main

import (
   "encoding/xml"
   "fmt"
   "io/ioutil"
)

// 抽取单个server对象
type Server struct {
   ServerName string `xml:"serverName"`
   ServerIP   string `xml:"serverIP"`
}

type Servers struct {
   Name    xml.Name `xml:"servers"`
   Version int   `xml:"version"`
   Servers []Server `xml:"server"`
}

func main() {
   data, err := ioutil.ReadFile("D:/my.xml")
   if err != nil {
      fmt.Println(err)
      return
   }
   var servers Servers
   err = xml.Unmarshal(data, &servers)
   if err != nil {
      fmt.Println(err)
      return
   }
   fmt.Printf("xml: %#v\n", servers)
}
```

### MSGPack

- MSGPack是二进制的json，性能更快，更省空间
- 需要安装第三方包：go get -u github.com/vmihailenco/msgpack

```go
package main

import (
   "fmt"
   "github.com/vmihailenco/msgpack"
   "io/ioutil"
   "math/rand"
)

type Person struct {
   Name string
   Age  int
   Sex  string
}

// 二进制写出
func writerJson(filename string) (err error) {
   var persons []*Person
   // 假数据
   for i := 0; i < 10; i++ {
      p := &Person{
         Name: fmt.Sprintf("name%d", i),
         Age:  rand.Intn(100),
         Sex:  "male",
      }
      persons = append(persons, p)
   }
   // 二进制json序列化
   data, err := msgpack.Marshal(persons)
   if err != nil {
      fmt.Println(err)
      return
   }
   err = ioutil.WriteFile(filename, data, 0666)
   if err != nil {
      fmt.Println(err)
      return
   }
   return
}

// 二进制读取
func readJson(filename string) (err error) {
   var persons []*Person
   // 读文件
   data, err := ioutil.ReadFile(filename)
   if err != nil {
      fmt.Println(err)
      return
   }
   // 反序列化
   err = msgpack.Unmarshal(data, &persons)
   if err != nil {
      fmt.Println(err)
      return
   }
   for _, v := range persons {
      fmt.Printf("%#v\n", v)
   }
   return
}

func main() {
   //err := writerJson("D:/person.dat")
   //if err != nil {
   // fmt.Println(err)
   // return
   //}
   err := readJson("D:/person.dat")
   if err != nil {
      fmt.Println(err)
      return
   }
}
```