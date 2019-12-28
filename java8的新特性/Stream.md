# 简介
Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对
集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。
使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数
据库查询。也可以使用 Stream API 来并行执行操作。简而言之，
Stream API 提供了一种高效且易于使用的处理数据的方式。
  
流(Stream) 到底是什么呢？ 
是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。
`集合讲的是数据，流讲的是计算！`
可以这样理解, 数据源（集合，数组等）通过一定的方法转化为流对象，这个流对象可以做很多的操作（过滤，分片等等），各种中间操作后得到一个新的数据源（我们想要的结果）。看下面的内容，慢慢理解吧!

# 操作过程
Stream 的操作分为3个步骤，分别是`创建Stream` `中间操作` `终止操作（终端操作）`,下面分别看这些是什么东西。
## 创建Stream
1. 通过集合的扩展方法创建
Java8中的Collection的接口被扩展，提供了两个获取流的方法。
```
//创建一个顺序流
default Stream<E> stream() {
            return StreamSupport.stream(spliterator(), false);
}

//创建一个并行流
default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
}

```
例子：
```
//1. Collection 提供了两个方法  stream() 与 parallelStream()
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream(); //获取一个顺序流
Stream<String> parallelStream = list.parallelStream(); //获取一个并行流
```

2. 由数组创建流
Java8中Arrays的静态方法`stream()`可以获取数组流
```
public static <T> Stream<T> stream(T[] array) {
            return stream(array, 0, array.length);
}

```
里面有很多不同重载的方法使用，例如：
```
public static IntStream stream(int[] array) {
            return stream(array, 0, array.length);
}

public static LongStream stream(long[] array) {
            return stream(array, 0, array.length);
}
```
更多去看`Arrays`这个里面的实现

3. 由值创建
可以使用Stream 类中静态方法 of(),通过显示的值创建一个流，接受任意数量级的参数。
```
public static<T> Stream<T> of(T t) {
            return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}

@SuppressWarnings("varargs") // Creating a stream from an array is safe
public static<T> Stream<T> of(T... values) {
            return Arrays.stream(values);
}
```
例如：
```
Stream<Integer> stream = Stream.of(1,2,3,4,5,6);
```

4. 创建无限流
可以使用静态方法`Stream.iterate()`和`Stream.generate()`,创建无限流。
* 通过迭代的方式
```
public static<T> Stream<T> iterate(final T seed, final 
UnaryOperator<T> f)
```
例如：
```
Stream<Integer> stream3 = Stream.iterate(0, (x) -> x + 2).limit(3);
stream3.forEach(System.out::println);
```
结果：
```
0
2
4
```
说明：这个会`.limit(3)`也就是中间操作，限制了3次，这里也是为了演示方便，如果不加，就会一直在输出，`.forEach(System.out::println)`是遍历结果。

* 通过生成的方式

```
Stream<Double> stream = Stream.generate(Math::random).limit(2);
stream.forEach(System.out::println);
```
结果：
```
0.7431402118053324
0.43342562634143633
```
`.limit(2)`这里限制生成两个，不加会无限生成