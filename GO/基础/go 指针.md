# 简介

区别于C/C++中的指针，Go语言中的指针不能进行偏移和运算，是安全指针。



每个变量在运行时都拥有一个地址，这个地址代表变量在内存中的位置。

- `&`（取地址）

- `*`（根据地址取值）

每个变量在运行时都拥有一个地址，这个地址代表变量在内存中的位置。Go语言中使用&字符放在变量前面对变量进行“取地址”操作。 Go语言中的值类型`（int、float、bool、string、array、struct）`都有对应的指针类型，如：`*int、*int64、*string`等。



- 指针地址：通过& 可以获取变量的地址。
- 指针类型: *int 也就可以理解位存放指向int类型的一个地址。
- 指针取值：通过&，可以获取地址，在通过*，就可以获取该地址的值。



## 空指针

- 当一个指针被定义后没有分配到任何变量时，它的值为 nil

- 空指针的判断

  ```go
  var p *string
  fmt.Println(p)
  fmt.Printf("p的值是%v\n", p)
  if p != nil {
      fmt.Println("非空")
  } else {
      fmt.Println("空值")
  }
  ```

  