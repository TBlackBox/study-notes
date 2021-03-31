# 简介

Go语言中没有“类”的概念，也不支持“类”的继承等面向对象的概念。Go语言中通过结构体的内嵌再配合接口比面向对象具有更高的扩展性和灵活性。



## 类型定义和类型别名

1. 类型定义

```go
//将MyInt定义为int类型
type MyInt int
//自定义类型是定义了一个全新的类型。我们可以基于内置的基本类型定义，也可以通过struct定义。
//通过Type关键字的定义，MyInt就是一种新的类型，它具有int的特性。
```

2. 类型别名

   ```go
   type TypeAlias = Type
   //别名  指向的是同一个类型 只是不同的名字或者叫法，编译完成后都不会有TypeAlias该类型了
   //byte 和rune 都是类型别名
   type byte = uint8
   type rune = int32
   ```

   例子

   ```go
   //类型定义
   type NewInt int
   
   //类型别名
   type MyInt = int
   
   func main() {
       var a NewInt
       var b MyInt
   
       fmt.Printf("type of a:%T\n", a) //type of a:main.NewInt
       fmt.Printf("type of b:%T\n", b) //type of b:int
   }
   //结果显示a的类型是main.NewInt，表示main包下定义的NewInt类型。b的类型是int。
   //MyInt类型只会在代码中存在，编译完成时并不会有MyInt类型。
   ```

   