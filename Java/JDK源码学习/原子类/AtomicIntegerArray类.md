# 简介


# 构造方法
```
private final int[] array;
public AtomicIntegerArray(int length) {
	array = new int[length];
}

public AtomicIntegerArray(int[] array) {
    // Visibility guaranteed by final field guarantees
    this.array = array.clone();
}
```

这里的构造方法也很简单，一种是传入数组的长度，默认创建一个length长度的各个元素为0的数组，一种是直接在外面创建好一个数组，然后传给构造方法。

这里保存数组的值是全局维护了一个int[] array. 有没有发现这里和AtomicInteger的不同？AtomicInteger保存值是维护了一个volatile来保证可见性，这里为什么没有采取同样的方法？

仔细看看一下，**array使用的是final修饰，变成了常量数组，引用不可变，这个array数组就保存到了方法区，同样的可以保证多线程访问时的可见性**，避免使用volatile也减少了开销。



# 简单用法
```
int[] intArray = new int[]{1, 2, 3, 4, 5};
AtomicIntegerArray aia = new AtomicIntegerArray(intArray);
aia.set(0, 5);
aia.getAndDecrement(0); // 5
aia.decrementAndGet(1); // 1
aia.getAndIncrement(2); // 3
aia.incrementAndGet(3); // 5
aia.addAndGet(0, 10); // 14
aia.getAndAdd(1, 10); // 1
aia.compareAndSet(2, 3, 10); // false
aia.get(2); // 4
aia.compareAndSet(2, 4, 10); // true
aia.get(2); // 10
```

# base和shift变量
```
 private static final int base = unsafe.arrayBaseOffset(int[].class);
 private static final int shift;

static {
    int scale = unsafe.arrayIndexScale(int[].class);
    if ((scale & (scale - 1)) != 0)
    throw new Error("data type scale not a power of two");
    shift = 31 - Integer.numberOfLeadingZeros(scale);
}
```
`arrayBaseOffset`和`arrayIndexScale`是`unsafe`的两个本地方法。
这两个native方法需要传入的参数都是一个array类型的class。
- BaseOffset是能获取数组首个元素的首地址偏移。
- rrayIndexScale可以用来获取数组元素的增量地址的方法。