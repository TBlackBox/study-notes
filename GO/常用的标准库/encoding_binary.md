# 简介

此包实现了对数据与byte之间的转换，以及varint的编解码。

数字通过读取和写入固定大小的值进行翻译。固定大小的值可以是固定大小的算术类型（bool，int8，uint8，int16，float32，complex64，...），也可以是只包含固定大小值的数组或结构。

varint函数使用可变长度编码对单个整数值进行编码和解码; 较小的值需要较少的字节。有关规范，请参阅https://developers.google.com/protocol-buffers/docs/encoding。

此包倾向于简化而不是效率。需要高性能序列化的客户端，尤其是大型数据结构的客户端，应该考虑更高级的解决方案，如binary/ gob包或协议缓冲区。


# 变量

BigEndian是ByteOrder的大端实现。

```go
var BigEndian bigEndian
```

LittleEndian是ByteOrder的小端实现。

```go
var LittleEndian littleEndian
```


# 结构体

```go
type ByteOrder：可以定义自己的字节序结构，用于序列化和反序列化数据。
```

# 数据的byte序列化转换

```go
func Read(r io.Reader, order ByteOrder, data interface{}) error
func Write(w io.Writer, order ByteOrder, data interface{}) error
func Size(v interface{}) int
```

###  func Read(r io.Reader, order ByteOrder, data interface{}) error

```go
参数列表：
1）r  可以读出字节流的数据源
2）order  特殊字节序，包中提供大端字节序和小端字节序
3）data  需要解码成的数据
返回值：error  返回错误
功能说明：Read从r中读出字节数据并反序列化成结构数据。data必须是固定长的数据值或固定长数据的slice。从r中读出的数据可以使用特殊的 字节序来解码，并顺序写入value的字段。当填充结构体时，使用(_)名的字段讲被跳过。

//案列
func main() {
    var pi float64
    b := []byte{0x18,0x2d,0x44,0x54,0xfb,0x21,0x09,0x40}
    buf := bytes.NewBuffer(b)
    err := binary.Read(buf, binary.LittleEndian, &pi)
    if err != nil {
        log.Fatalln("binary.Read failed:", err)
    }
    fmt.Println(pi)
}
```

### func Write(w io.Writer, order ByteOrder, data interface{}) error

```go
参数列表：
1）w  可写入字节流的数据
2）order  特殊字节序，包中提供大端字节序和小端字节序
3）data  需要解码的数据
返回值：error  返回错误
功能说明：
Write讲data序列化成字节流写入w中。data必须是固定长度的数据值或固定长数据的slice，或指向此类数据的指针。写入w的字节流可用特殊的字节序来编码。另外，结构体中的(_)名的字段讲忽略。

//案列
buf := new(bytes.Buffer)
pi := math.Pi

err := binary.Write(buf, binary.LittleEndian, pi)
if err != nil {
    log.Fatalln(err)
}
fmt.Println(buf.Bytes())
```

### func Size(v interface{}) int

```go
参数列表：v  需要计算长度的数据
返回值：int 数据序列化之后的字节长度
功能说明：
Size将返回数据系列化之后的字节长度，数据必须是固定长数据类型、slice和结构体及其指针。

//案列
func main() {
    var a int
    p := &a
    b := [10]int64{1}
    s := "adsa"
    bs := make([]byte, 10)

    fmt.Println(binary.Size(a)) // -1
    fmt.Println(binary.Size(p)) // -1
    fmt.Println(binary.Size(b)) // 80
    fmt.Println(binary.Size(s)) // -1
    fmt.Println(binary.Size(bs))    // 10
}
```



# uvarint和varint的编解码

```go
func PutUvalint(buf []byte, x uint64) int
func PutVarint(buf []byte, x int64) int
func Uvarint(buf []byte) (uint64, int)
func Varint(buf []byte) (int64, int)
func ReadUvarint(r io.ByteReader) (uint64, error)
func ReadVarint(r io.ByteReader) (int64, error)
```

### func PutUvalint(buf []byte, x uint64) int

```go
参数列表：
1）buf  需写入的缓冲区
2）x  uint64类型数字
返回值：
1）int  写入字节数。
2）panic  buf过小。
功能说明：
PutUvarint主要是将uint64类型放入buf中，并返回写入的字节数。如果buf过小，PutUvarint将抛出panic。

//案列
func main() {
    u16 := 1234
    u64 := 0x1020304040302010
    sbuf := make([]byte, 4)
    buf := make([]byte, 10)

    ret := binary.PutUvarint(sbuf, uint64(u16))
    fmt.Println(ret, len(strconv.Itoa(u16)), sbuf)

    ret = binary.PutUvarint(buf, uint64(u64))
    fmt.Println(ret, len(strconv.Itoa(u64)), buf)
}
/*
输出结果：
2 4 [210 9 0 0]
9 19 [144 192 192 129 132 136 140 144 16 0]
会发现转成二进制来传输数据，比直接转字符串之后转[]byte这种方式传更节省传输空间
 */
```

### func PutVarint(buf []byte, x int64) int

```go

参数列表：
1）buf  需要写入的缓冲区
2）x int64类型数字
返回值：
1）int  写入字节数
2）panic  buf过小
功能说明：
PutVarint主要是讲int64类型放入buf中，并返回写入的字节数。如果buf过小，PutVarint将抛出panic。

//案列
func main() {
    i16 := 1234
    i64 := -1234567890
    sbuf := make([]byte, 4)
    buf := make([]byte, 10)

    ret := binary.PutVarint(buf, int64(i16))
    fmt.Println(ret, len(strconv.Itoa(i16)), sbuf)

    ret = binary.PutVarint(buf, int64(i64))
    fmt.Println(ret, len(strconv.Itoa(i64)), buf)
}
/*
2 4 [0 0 0 0]
5 11 [163 139 176 153 9 0 0 0 0 0]
 */
```

### func Uvarint(buf []byte) (uint64, int)

```go
参数列表：buf  需要解码的缓冲区
返回值：
1）uint64  解码的数据。
2）int  解析的字节数。
功能说明:
Uvarint是从buf中解码并返回一个uint64的数据,及解码的字节数(>0)。如果出错,则返回数据0和一个小于等于0的字节数n,其意义为:
1)n == 0: buf太小
2)n < 0: 数据太大,超出uint64最大范围,且-n为已解析字节数

//案列
func main() {
    sbuf := []byte{}
    buf := []byte{144,192,192,132,136,140,144,16,0,1,1}
    bbuf := []byte{144,192,192,129,132,136,140,144,192,192,1,1}

    num, ret := binary.Uvarint(sbuf)
    fmt.Println(num, ret)

    num, ret = binary.Uvarint(buf)
    fmt.Println(num, ret)

    num, ret = binary.Uvarint(bbuf)
    fmt.Println(num, ret)
}
```

###  func Varint(buf []byte) (int64, int)
```go
参数列表: buf  需要解码的缓冲区
返回值:
1) int64 解码的数据
2) int  解析的字节数
功能说明:
Varint是从buf中解码并返回一个int64的数据,及解码的字节数(>0).如果出错,则返回数据0和一个小于等于0的字节数n,其意义为:
1) n == 0: buf太小
2) n < 0: 数据太大,超出64位,且-n为已解析字节数

//案列
func main() {
    var sbuf []byte
    var buf []byte = []byte{144, 192, 192, 129, 132, 136, 140, 144, 16, 0, 1, 1}
    var bbuf []byte = []byte{144, 192, 192, 129, 132, 136, 140, 144, 192, 192, 1, 1}

    num, ret := binary.Varint(sbuf)
    fmt.Println(num, ret) //0 0

    num, ret = binary.Varint(buf)
    fmt.Println(num, ret) //580990878187261960 9

    num, ret = binary.Varint(bbuf)
    fmt.Println(num, ret) //0 -11
}
```

### func ReadUvarint(r io.ByteReader) (uint64, error)
```go
参数列表:
返回值:
1) uint64  解析出的数据
2) error  返回的错误
功能说明:
ReadUvarint从r中解析并返回一个uint64类型的数据及出现的错误.


//案列
func main() {
    var sbuf []byte
    var buf []byte = []byte{144, 192, 192, 129, 132, 136, 140, 144, 16, 0, 1, 1}
    var bbuf []byte = []byte{144, 192, 192, 129, 132, 136, 140, 144, 192, 192, 1, 1}

    num, err := binary.ReadUvarint(bytes.NewBuffer(sbuf))
    fmt.Println(num, err) //0 EOF

    num, err = binary.ReadUvarint(bytes.NewBuffer(buf))
    fmt.Println(num, err) //1161981756374523920 <nil>

    num, err = binary.ReadUvarint(bytes.NewBuffer(bbuf))
    fmt.Println(num, err) //4620746270195064848 binary: varint overflows a 64-bit integer
}

```

### func ReadVarint(r io.ByteReader) (int64, error)

```go
参数列表: r  实现ByteReader接口的对象
返回值: 
1) int64  解析出的数据
2) error  返回的错误
功能说明:
ReadVarint从r中解析并返回一个int64类型的数据及出现的错误.

func main() {
    var sbuf []byte
    var buf []byte = []byte{144, 192, 192, 129, 132, 136, 140, 144, 16, 0, 1, 1}
    var bbuf []byte = []byte{144, 192, 192, 129, 132, 136, 140, 144, 192, 192, 1, 1}

    num, err := binary.ReadVarint(bytes.NewBuffer(sbuf))
    fmt.Println(num, err) //0 EOF

    num, err = binary.ReadVarint(bytes.NewBuffer(buf))
    fmt.Println(num, err) //580990878187261960 <nil>

    num, err = binary.ReadVarint(bytes.NewBuffer(bbuf))
    fmt.Println(num, err) //2310373135097532424 binary: varint overflows a 64-bit integer
}
```


# 编解码的几种方法

1. 编码

```go
func Write(w io.Writer, order ByteOrder, data interface{}) error
func PutUvarint(buf []byte, x uint64) int
func PutVarint(buf []byte, x int64) int
```

2. 解析

   ```go
   func Uvarint(buf []byte) (uint64, int)
   func Varint(buf []byte) (int64, int)
   func ReadUvarint(r io.ByteReader) (uint64, error)//从r中解析并返回一个uint64类型的数据及出现的错误.
   func ReadVarint(r io.ByteReader) (int64, error) //从r中解析并返回一个int64类型的数据及出现的错误.
   ```

   

