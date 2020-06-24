# 简介
提供了可以原子的读写对象引用的一种机制。

是AtomicReferenceXXX相关类的最基础类，还有AtomicReferenceArray、AtomicReferenceFieldUpdater、AtomicStampedReference等。

- 使用CAS操作保证原子性, 使用volatile修饰保证可见性;
- 相等的比较算法是 ==,即基本数据类型判断值，对象类型判断引用地址是否相同;

## 注意
AtomicReferenceArray: 初始化时计算数组的起始位置和每个元素的偏移量,通过base + i << shift进行数组索引;
AtomicReferenceFieldUpdater: 对指定类中的volatile属性进行原子更新，一般使用不会存在问题。但如果涉及自定义类加载器的场景，需要注意需保证在同一组类加载器加载中。


# 接口列表
```
// 构造初始值为null的AtomicReference对象
public AtomicReference();

// 构造初始值为initialValue的AtomicReference对象
public AtomicReference(V initialValue)

// 获取当前value对象
public final V get()

// 直接更新volatile对象的值
public final void set(V newValue)

//设置了要过小段时间才会生效，不会立即生效
public final void lazySet(V newValue)

// 基于 == 操作进行的CAS更新
public final boolean compareAndSet(V expect, V update)

// 高效但不保证可见性的原子set操作，在无happens-before需求的场景下使用
public final boolean weakCompareAndSet(V expect, V update)

// 设置给定值,并返回旧值
public final V getAndSet(V newValue)

// 通过可重入无状态的方法更新对象值,并返回旧值
// 内部通过get + CAS自旋的策略进行
public final V getAndUpdate(UnaryOperator<V> updateFunction)

// 与上一个方法类似,只是返回更新后的新值
public final V updateAndGet(UnaryOperator<V> updateFunction)

// 回调方法可以带入参的更新操作,返回旧值
// V x作为accumulatorFunction的一个入参,accumulatorFunction.apply(prev, x)
public final V getAndAccumulate(V x,BinaryOperator<V> accumulatorFunction)

// 与上一个方法类似,返回新值
public final V accumulateAndGet(V x,BinaryOperator<V> accumulatorFunction)
```



# 原子更新属性

AtomicReference系列可以更新Object，同样的，针对Object的属性，Atomic提供一下方法来更新Object的属性：

1. AtomicIntegerFieldUpdater：用于更新Object的整型属性；

2. AtomicLongFieldUpdater：用于更新Object的长整型属性；

3. AtomicReferenceFieldUpdater：用于更新Object的引用类型属性。
