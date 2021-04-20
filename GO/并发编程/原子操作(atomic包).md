###  原子操作

代码中的加锁操作因为涉及内核态的上下文切换会比较耗时、代价比较高。针对基本数据类型我们还可以使用原子操作来保证并发安全，因为原子操作是Go语言提供的方法它在用户态就可以完成，因此性能比加锁操作更好。Go语言中原子操作由内置的标准库sync/atomic提供。



### atomic包

| 方法                                                         | 解释           |
| ------------------------------------------------------------ | -------------- |
| func LoadInt32(addr *int32) (val int32) <br>func LoadInt64(addr `*int64`) (val int64)<br>func LoadUint32(addr`*uint32`) (val uint32)<br>func LoadUint64(addr`*uint64`) (val uint64)<br>func LoadUintptr(addr`*uintptr`) (val uintptr)<br>func LoadPointer(addr`*unsafe.Pointer`) (val unsafe.Pointer) | 读取操作       |
| func StoreInt32(addr `*int32`, val int32) <br>func StoreInt64(addr `*int64`, val int64) <br>func StoreUint32(addr `*uint32`, val uint32) <br>func StoreUint64(addr `*uint64`, val uint64) <br>func StoreUintptr(addr `*uintptr`, val uintptr) <br>func StorePointer(addr `*unsafe.Pointer`, val unsafe.Pointer) | 写入操作       |
| func AddInt32(addr `*int32`, delta int32) (new int32) <br>func AddInt64(addr `*int64`, delta int64) (new int64)<br> func AddUint32(addr `*uint32`, delta uint32) (new uint32) <br>func AddUint64(addr `*uint64`, delta uint64) (new uint64) <br>func AddUintptr(addr `*uintptr`, delta uintptr) (new uintptr) | 修改操作       |
| func SwapInt32(addr `*int32`, new int32) (old int32) <br>func SwapInt64(addr `*int64`, new int64) (old int64) <br>func SwapUint32(addr `*uint32`, new uint32) (old uint32) <br>func SwapUint64(addr `*uint64`, new uint64) (old uint64) <br>func SwapUintptr(addr `*uintptr`, new uintptr) (old uintptr) <br>func SwapPointer(addr `*unsafe.Pointer`, new unsafe.Pointer) (old unsafe.Pointer) | 交换操作       |
| func CompareAndSwapInt32(addr `*int32`, old, new int32) (swapped bool) <br>func CompareAndSwapInt64(addr `*int64`, old, new int64) (swapped bool) <br>func CompareAndSwapUint32(addr `*uint32`, old, new uint32) (swapped bool) <br>func CompareAndSwapUint64(addr `*uint64`, old, new uint64) (swapped bool) <br>func CompareAndSwapUintptr(addr `*uintptr`, old, new uintptr) (swapped bool) <br>func CompareAndSwapPointer(addr `*unsafe.Pointer`, old, new unsafe.Pointer) (swapped bool) | 比较并交换操作 |

###  示例

```go
var x int64
var l sync.Mutex
var wg sync.WaitGroup

// 普通版加函数
func add() {
    // x = x + 1
    x++ // 等价于上面的操作
    wg.Done()
}

// 互斥锁版加函数
func mutexAdd() {
    l.Lock()
    x++
    l.Unlock()
    wg.Done()
}

// 原子操作版加函数
func atomicAdd() {
    atomic.AddInt64(&x, 1)
    wg.Done()
}

func main() {
    start := time.Now()
    for i := 0; i < 10000; i++ {
        wg.Add(1)
        // go add()       // 普通版add函数 不是并发安全的
        // go mutexAdd()  // 加锁版add函数 是并发安全的，但是加锁性能开销大
        go atomicAdd() // 原子操作版add函数 是并发安全，性能优于加锁版
    }
    wg.Wait()
    end := time.Now()
    fmt.Println(x)
    fmt.Println(end.Sub(start))
}
```

atomic包提供了底层的原子级内存操作，对于同步算法的实现很有用。这些函数必须谨慎地保证正确使用。除了某些特殊的底层应用，使用通道或者sync包的函数/类型实现同步更好。