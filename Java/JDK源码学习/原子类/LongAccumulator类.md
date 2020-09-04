# 简介
`LongAccumulator`类和`longAdder`一样继承`Striped64`，可以看做`longAdder`只是`LongAccumulator`的一个特列。LongAccumulator提供了比LongAdder更强大的功能，如下构造函数其中accumulatorFunction一个双目运算器接口，根据输入的两个参数返回一个计算值，identity则是LongAccumulator累加器的初始值。

# 构造器
```
public LongAccumulator(LongBinaryOperator accumulatorFunction,
long identity) {
    this.function = accumulatorFunction;
    base = this.identity = identity;
}
```
## LongBinaryOperator接口
```
@FunctionalInterface
public interface LongBinaryOperator {
    long applyAsLong(long left, long right);
}
```

LongAdder其实是LongAccumulator的一个特例，调用LongAdder相当使用下面的方式调用LongAccumulator。

```
LongAdder adder = new LongAdder();
```
```
    LongAccumulator accumulator = new LongAccumulator(new LongBinaryOperator() {
        
        @Override
        public long applyAsLong(long left, long right) {
            return left + right;
        }
    }, 0);
```
LongAccumulator相比于LongAdder可以提供累加器初始非0值，后者只能默认为0，另外前者还可以指定累加规则比如不是累加而是相乘，只需要构造LongAccumulator时候传入自定义双面运算器就OK，后者则内置累加的规则。

从下面代码知道LongAccumulator相比于LongAdder不同在于casBase时候后者传递的是b+x,前者则是调用了r = function.applyAsLong(b = base, x)来计算。
```
  public void add(long x) {
        Cell[] as; long b, v; int m; Cell a;
        if ((as = cells) != null || !casBase(b = base, b + x)) {
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[getProbe() & m]) == null ||
                !(uncontended = a.cas(v = a.value, v + x)))
                longAccumulate(x, null, uncontended);
        }
    }
```

```
    public void accumulate(long x) {
        Cell[] as; long b, v, r; int m; Cell a;
        if ((as = cells) != null ||
            (r = function.applyAsLong(b = base, x)) != b && !casBase(b, r)) {
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[getProbe() & m]) == null ||
                !(uncontended =
                  (r = function.applyAsLong(v = a.value, x)) == v ||
                  a.cas(v, r)))
                longAccumulate(x, function, uncontended);
        }
    }
```
另外前者调用longAccumulate时候传递到是function,而后者是null，从下面代码可知当fn为null时候就是使用v+x加法运算这时候就等价于LongAdder，fn不为null时候则使用传递的fn函数计算，如果fn为加法则等价于LongAdder；
```
  else if (casBase(v = base, ((fn == null) ? v + x :
                                        fn.applyAsLong(v, x))))
                break;                          // Fall back on using base
```

# 总结
1. `longAdder`可以看做是`LongAccumulate`的一个`+`的特列，LongAccumulate更加的灵活，可以通过函数式接口`LongBinaryOperator`定制你想要得代码

2. 同理，`DoubleAdder`和`DoubleAccumulator`更上面的差不多，只是数据类型是double，都不做过多的讲解了。
