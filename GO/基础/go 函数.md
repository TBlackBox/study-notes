# 定义

## 特点

```
• 无需声明原型。
• 支持不定 变参。
• 支持多返回值。
• 支持命名返回参数。 
• 支持匿名函数和闭包。
• 函数也是一种类型，一个函数可以赋值给变量。

• 不支持 嵌套 (nested) 一个包不能有两个名字一样的函数。
• 不支持 重载 (overload) 
• 不支持 默认参数 (default parameter)
```
## 声明
```go
func test(x, y int, s string) (int, string) {
    // 类型相同的相邻参数，参数类型可合并。 多返回值必须用括号。
    n := x + y          
    return n, fmt.Sprintf(s, n)
}

函数声明包含一个函数名，参数列表， 返回值列表和函数体。如果函数没有返回值，则返回列表可以省略。函数从第一条语句开始执行，直到执行return语句或者执行函数的最后一条语句。
函数可以没有参数或接受多个参数。
注意类型在变量名之后 。
当两个或多个连续的函数命名参数是同一类型，则除了最后一个类型之外，其他都可以省略。
函数可以返回任意数量的返回值。
使用关键字 func 定义函数，左大括号依旧不能另起一行。
```

函数是第一类对象，可作为参数传递。建议将复杂签名定义为函数类型，以便于阅读。

```go
func test(fn func() int) int {
    return fn()
}
// 定义函数类型。
type FormatFunc func(s string, x, y int) string 

func format(fn FormatFunc, s string, x, y int) string {
    return fn(s, x, y)
}

func main() {
    s1 := test(func() int { return 100 }) // 直接将匿名函数当参数。

    s2 := format(func(s string, x, y int) string {
        return fmt.Sprintf(s, x, y)
    }, "%d, %d", 10, 20)

    println(s1, s2)
}

//输出
100 10, 20
```

有返回值的函数，必须有明确的终止语句，否则会引发编译错误。

你可能会偶尔遇到没有函数体的函数声明，这表示该函数不是以Go实现的。这样的声明定义了函数标识符。

```go
package math

func Sin(x float64) float //implemented in assembly language
```



# 参数

函数的参数，该变量称为形参，相当于函数的局部变量，传过来的变量就是实参，两种传递方式：

- 值传递：将参数复制一份到函数，函数中修改参数，不会影响实际参数
- 引用传递：就是串实际产生的地址，函数中修改会影响实际参数

默认情况下是值传递，不会影响实际参数，但需要注意：

- 无论是值传递还是引用传递，传递给函数的都是变量的副本，不过，值传递是拷贝（大对像，效率很低），引用时拷贝地址（更高效）

- map,slice,chan,指针，interface默认时引用传递.

- 可变参数：本质时slice,只能有一个，且必须时最后一个，在参数赋值时可以不用用一个一个的赋值，可以直接传递一个数组或者切片，特别注意的是在参数后加上“…”即可

  ```go
  func myfunc(args ...int) {    //0个或多个参数
  }
  
  func add(a int, args…int) int {    //1个或多个参数
  }
  
  func add(a int, b int, args…int) int {    //2个或多个参数
  }
  //注意：其中args是一个slice，我们可以通过arg[index]依次访问所有参数,通过len(arg)来判断传递参数的个数.
  //slice 对象做变参时，必须展开。
  s := []int{1, 2, 3}
  myfuncI(s...)
  ```

  任意类型的不定参数： 就是函数的参数和每个参数的类型都不是固定的。

  用interface{}传递任意类型数据是Go语言的惯例用法，而且interface{}是类型安全的。

  ```go
  func myfunc(args ...interface{}) {
    }
  ```

# 返回值

`"_"`标识符，用来忽略函数的某个返回值

返回值的名称应当具有一定的意义，可以作为文档使用。

**“裸”返回**

```go
//没有参数的 return 语句返回各个返回变量的当前值。这种用法被称作“裸”返回。
func add(a, b int) (c int) {
    c = a + b
    return
}

func calc(a, b int) (sum int, avg int) {
    sum = a + b
    avg = (a + b) / 2

    return
}
//直接返回语句仅应当用在像下面这样的短函数中。在长的函数中它们会影响代码的可读性。
```

回值不能用容器对象接收多返回值。只能用多个变量，或 `"_"` 忽略。

```go
func test() (int, int) {
    return 1, 2
}

func main() {
    // s := make([]int, 2)
    // s = test()   // Error: multiple-value test() in single-value context

    x, _ := test()
    println(x)
}
```

多返回值可以作为其他函数的参数调用。

命名返回参数可看做与形参类似的局部变量，最后由 return 隐式返回。

```go
func add(x, y int) (z int) {
    z = x + y
    return
    //var z = x + y   命名返回参数可被同名局部变量遮蔽，此时需要显式返回。
   // return z
}

func main() {
    println(add(1, 2)) //3
}
```

命名返回参数允许 defer 延迟调用通过闭包读取和修改。

```go

func add(x, y int) (z int) {
    defer func() {
        z += 100
    }()

    z = x + y
    return
}

func main() {
    println(add(1, 2))  //103
}
```

显式 return 返回前，会先修改命名返回参数。

```go
func add(x, y int) (z int) {
    defer func() {
        println(z) // 输出: 203
    }()

    z = x + y
    return z + 200 // 执行顺序: (z = z + 200) -> (call defer) -> (return)
}

func main() {
    println(add(1, 2)) // 输出: 203
}
//输出
203
203
```



