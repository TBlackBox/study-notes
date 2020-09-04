# 简介
Java 5 中由Doug Lea大神写的atomic classes 中引入了 Field Updater，本质上来说就是volatile 字段的包装器。`AtomicIntegerFieldUpdater<T>`是一个抽象类，主要用于原子更新对象里面的属性。他的实现类`AtomicIntegerFieldUpdaterImpl`都在他的内部。

# newUpdater（）获取实例
```
private static AtomicIntegerFieldUpdater<Test> update = AtomicIntegerFieldUpdater.newUpdater(Test.class, "a");
```
注意：a是测试类里面的一个属性。
```
public volatile int a = 100;
```

# 总结
1. 使用场景不多，有点类似AtomicInteger,但是可以使用比较方便，比如定义了一个变量x,你可以直接正常访问x，而如果使用了AtomicInteger的话，还是需要调用AtomicInteger#get方法，另外如果你又需要原子的get-set的话也可以满足使用场景；另外比如说你有个比较大的链表，内部每个节点都需要原子的get-set，那每个节点都要定义一个AtomicInteger，这样会消耗更多的内存，那么就可以定义个static的AtomicIntegerFieldUpdater。

