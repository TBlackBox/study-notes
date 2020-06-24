# 简介
AtomicLong；用来更新长整形的变量。基本 用法跟AtomicInteger是一样的。



# 用于序列号生成器
jdk的文档说AtomicLong是一个代表能别原子化更新的long值。AtomicLong可以在应用中作为一个序列号生成器来使用。
```
AtomicLong seqNum = new AtomicLong(1);
long id = seqNum.getAndIncrement();
```

# lazySet方法
lazySet方法调用Unsafe的putOrderedLong方法将value设置成newValue，但是使用lazySet设置值可能会导致其他线程在之后的一小段时间内读到的值还是设置之前的旧值，Hotspot的unsafe具体实现中给出有原因。



