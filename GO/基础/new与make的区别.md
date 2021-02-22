# 简介
new 和make 都是go的内置函数，都是用来分配内存的。

### new 的函数前面
```go
func new(Type) *Type
```
### make 的函数前面
```go
func make(t Type, size ...IntegerType) Type
```

## 主要区别
1. 二者都是用来做内存分配的。
2. make只用于slice、map以及channel的初始化，返回的还是这三个引用类型本身；
3. 而new用于类型的内存分配，并且内存对应的值为类型零值，返回的是指向类型的指针。