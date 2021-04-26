# 简介

map是一种无序的基于key-value的数据结构，Go语言中的map是引用类型，必须初始化才能使用。



最通俗的话说Map是一种通过key来获取value的一个数据结构，其底层存储方式为数组，在存储时key不能重复，当key重复时，value进行覆盖，我们通过key进行hash运算（可以简单理解为把key转化为一个整形数字）然后对数组的长度取余，得到key存储在数组的哪个下标位置，最后将key和value组装为一个结构体，放入数组下标处

```go
length = len(array) = 4
hashkey1 = hash(xiaoming) = 4
index1  = hashkey1% length= 0
hashkey2 = hash(xiaoli) = 6
index2  = hashkey2% length= 2
```

hash 冲突

- 有线性探测法： 冲突了，往后找，有空位就用
- 拉链法：冲突了，顺着冲突的链表往下找
- 线性补偿探测法
- 随机探测法



定义

```go
map[KeyType]ValueType

KeyType:表示键的类型。
ValueType:表示键对应的值的类型。

默认初始值位nil,需通过make()分配
make(map[KeyType]ValueType, [cap])
```



基本使用

```go
scoreMap := make(map[string]int, 8)
scoreMap["张三"] = 90
scoreMap["小明"] = 100
fmt.Println(scoreMap) //map[小明:100 张三:90]
fmt.Println(scoreMap["小明"]) //
fmt.Printf("type of a:%T\n", scoreMap) //type of a: map[string]int
```



判断是否存在

```go
value, ok := map[key]
// 如果key存在ok为true,v为对应的值；不存在ok为false,v为值类型的零值
```



遍历

```go
scoreMap := make(map[string]int)
scoreMap["张三"] = 90
scoreMap["小明"] = 100
scoreMap["王五"] = 60
//建 值都遍历
for k, v := range scoreMap {
    fmt.Println(k, v)
}
//只遍历key
for k := range scoreMap {
    fmt.Println(k)
}
//注意： 遍历map时的元素顺序与添加键值对的顺序无关。


//按照指定顺序遍历
rand.Seed(time.Now().UnixNano()) //初始化随机数种子

var scoreMap = make(map[string]int, 200)

for i := 0; i < 100; i++ {
    key := fmt.Sprintf("stu%02d", i) //生成stu开头的字符串
    value := rand.Intn(100)          //生成0~99的随机整数
    scoreMap[key] = value
}
//取出map中的所有key存入切片keys
var keys = make([]string, 0, 200)
for key := range scoreMap {
    keys = append(keys, key)
}
//对切片进行排序
sort.Strings(keys)
//按照排序后的key遍历map
for _, key := range keys {
    fmt.Println(key, scoreMap[key])
}
```



删除建值对

```go
delete(map, key)
map:表示要删除键值对的map
key:表示要删除的键值对的键
```



map大小

```go
size := len(map)
```



切片中元素为map

```go
var mapSlice = make([]map[string]string, 3)
for index, value := range mapSlice {
    fmt.Printf("index:%d value:%v\n", index, value)
}
fmt.Println("after init")
// 对切片中的map元素进行初始化
mapSlice[0] = make(map[string]string, 10)
mapSlice[0]["name"] = "王五"
mapSlice[0]["password"] = "123456"
mapSlice[0]["address"] = "红旗大街"
for index, value := range mapSlice {
    fmt.Printf("index:%d value:%v\n", index, value)
}
```



值为切片类型的map

```go
var sliceMap = make(map[string][]string, 3)
fmt.Println(sliceMap)
fmt.Println("after init")
key := "中国"
value, ok := sliceMap[key]
if !ok {
    value = make([]string, 0, 2)
}
value = append(value, "北京", "上海")
sliceMap[key] = value
fmt.Println(sliceMap)
```





## 原理

源码位于`src/runtime/map.go`包中

底层是数组，每个数组存储bucket,每个bucket存储8个键值对，到达8个键值对的时候，通过overflow指针指向一个新地址，从而形成一个链表

bucket的定义

```go
//bucket结构体定义 b就是bucket
type bmap{
    // tophash generally contains the top byte of the hash value
    // for each key  in this bucket. If tophash[0] < minTopHash,
    // tophash[0] is a bucket               evacuation state instead.
    //翻译：top hash通常包含该bucket中每个键的hash值的高八位。
    //如果tophash[0]小于mintophash，则tophash[0]为桶疏散状态    //bucketCnt 的初始值是8
    tophash [bucketCnt]uint8
    // Followed by bucketCnt keys and then bucketCnt values.
    // NOTE: packing all the keys together and then all the values together makes the    // code a bit more complicated than alternating key/value/key/value/... but it allows    // us to eliminate padding which would be needed for, e.g., map[int64]int8.// Followed by an overflow pointer.    //翻译：接下来是bucketcnt键，然后是bucketcnt值。
    //注意：将所有键打包在一起，然后将所有值打包在一起，    使得代码比交替键/值/键/值/更复杂。但它允许
    //我们消除可能需要的填充，    例如map[int64]int8./后面跟一个溢出指针}
```

tophash用来快速查找key值是否在该bucket中，而不同每次都通过真值进行比较；

还有kv的存放，为什么不是k1v1，k2v2..... 而是k1k2...v1v2...，我们看上面的注释说的 map[int64]int8,key是int64（8个字节），value是int8（一个字节），kv的长度不同，如果按照kv格式存放，则考虑内存对齐v也会占用int64，而按照后者存储时，8个v刚好占用一个int64,从这个就可以看出go的map设计之巧妙。

往map中存储一个kv对时，通过k获取hash值，hash值的低八位和bucket数组长度取余，定位到在数组中的那个下标，hash值的高八位存储在bucket中的tophash中，用来快速判断key是否存在，key和value的具体值则通过指针运算存储，当一个bucket满时，通过overfolw指针链接到下一个bucket。

