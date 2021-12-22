# 简介
Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。
使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来并行执行操作。简而言之，
Stream API 提供了一种高效且易于使用的处理数据的方式。

流(Stream) 到底是什么呢？ 
是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。
`集合讲的是数据，流讲的是计算！`
可以这样理解, 数据源（集合，数组等）通过一定的方法转化为流对象，这个流对象可以做很多的操作（过滤，分片等等），各种中间操作后得到一个**新的数据源**（我们想要的结果）。看下面的内容，慢慢理解吧!

# 操作过程
Stream 的操作分为3个步骤，分别是`创建Stream` `中间操作` `终止操作（终端操作）`,下面分别看这些是什么东西。
## 创建Stream
1. 通过集合的扩展方法创建Java8中的Collection的接口被扩展，提供了两个获取流的方法。
```java
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
```java
//1. Collection 提供了两个方法  stream() 与 parallelStream()
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream(); //获取一个顺序流
Stream<String> parallelStream = list.parallelStream(); //获取一个并行流
```

2. 由数组创建流
Java8中Arrays的静态方法`stream()`可以获取数组流
```java
public static <T> Stream<T> stream(T[] array) {
	return stream(array, 0, array.length);
}
```
里面有很多不同重载的方法使用，例如：
```java
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
```java
public static<T> Stream<T> of(T t) {
            return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}

@SuppressWarnings("varargs") // Creating a stream from an array is safe
public static<T> Stream<T> of(T... values) {
            return Arrays.stream(values);
}
```
例如：
```java
Stream<Integer> stream = Stream.of(1,2,3,4,5,6);
```

4. 创建无限流
可以使用静态方法`Stream.iterate()`和`Stream.generate()`,创建无限流。
* 通过迭代的方式
```java
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)
```
例如：
```java
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

```java
Stream<Double> stream = Stream.generate(Math::random).limit(2);
stream.forEach(System.out::println);
```
结果：
```
0.7431402118053324
0.43342562634143633
```
`.limit(2)`这里限制生成两个，不加会无限生成

# Stream的中间操作
多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理，而是在终止操作的时候一次性的全部处理，称为`惰性求值`
1. 筛选和切片
| 方法 | 描述 |
|----------|---------|
|filter(Predicate p)| 接受lambda,从流总排序排除某些元素。|
|distinct|刷选，通过流所生的的hashCode()和equals()去除重复的元素|
|limit(long MaxSize)| 截断流，使其元素不超过给定的数量|
|skip(long n)|跳过元素，返回一个扔掉了前n个元素的流。若流中的元素不足n个，则返回一个空流。与limit(n)互补|


2. 映射
| 方法 | 描述 |
|----------|---------|
|map(Function f)|接受一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素|
|mapToDouble(ToDoubleFunction f)|接受一个函数作为参数，该函数会被应用到每个元素上，产生一个新的DoubleStream,同理函数可以是`mapToInt` `mapToLong`|
|flatMap(Function f)|接收一个函数作为参数，将流中的每一个值都换成另外一个流，然后将所有流链接成一个流|

3. 排序
|方法|描述|
|---------|----------|
|sorted()|产生一个新流，其中按自然顺序排序|
|sorted(Comparatr comp)|产生一个新流，其中按照比较器的顺序排序|

```java
1、sorted() 默认使用自然序排序， 其中的元素必须实现Comparable 接口
2、sorted(Comparator<? super T> comparator) ：我们可以使用lambada 来创建一个Comparator 实例。可以按照升序或着降序来排序元素。

#自然序排序一个list
list.stream().sorted() 
 
#自然序逆序元素，使用Comparator 提供的reverseOrder() 方法
list.stream().sorted(Comparator.reverseOrder()) 
 
# 使用Comparator 来排序一个list
list.stream().sorted(Comparator.comparing(Student::getAge)) 
 
# 颠倒使用Comparator 来排序一个list的顺序，使用Comparator 提供的reverseOrder() 方法
list.stream().sorted(Comparator.comparing(Student::getAge).reversed()) 
```





# Stream的终止操作

终端操作会从流的流水线生产结果，其结果可以是任何不时流的值，例如：List,Integer,慎重void。下面看一下有哪些终端操作。
1. 查找与匹配
|方法|描述|
|--------|----------|
|allMatch(Predicate )|检查是否匹配所有元素|
|anyMatch(Predicate p)|检查是否至少匹配一个元素|
|noneMatch(Predicate p)|检查是否没有匹配所有元素|
|findFirst()|返回第一个元素|
|findAny()|返回当前流中的任意元素|
|count()|返回流中元素总数|
|max(Comparator c)|返回流中最大值|
|min(Comparatoc c)|返回流中最小值|
|forEach(Consumer c)|内部迭代（使用Collection接口需要用户去做迭代，称为外部部迭代，相反，Stream API 使用内部迭代——他帮你把迭代做了）|

2. 归约
|方法|描述|
|--------|----------|
|reduce(T iden,BinaryOperator b)|可以将流中的元素反复结合起来，得到一个值。返回 T,iden:为初始值，没有为空的情况|
|reduce(BinaryOperator b)|可以将流中的元素反复结合起来，得到一个值，返回Optional<T>,因为没有初始值，可能存在为空的情况，所有返回Optional<T>|

3. 收集
|方法|描述|
|---------|---------|
|collect(Collector c)|将流转化为其他形式，接受一个Collector接口的实现，用于给Dtream中的元素做汇总的方法|

`Collector`接口中的方法实现决定了如何对流执行收集操作(如收集到`List`,`Set`,`Map`)。Collectors适用类提供了很多的静态方法，可以方便地创建收集器实例。

*** 常用的说明 ***
为方便说明,构建下面的列表

```java
List<Employee> emps = Arrays.asList(
            new Employee(102, "李四", 79, 6666.66, Status.BUSY),
            new Employee(101, "张三", 18, 9999.99, Status.FREE),
            new Employee(103, "王五", 28, 3333.33, Status.VOCATION),
            new Employee(104, "赵六", 8, 7777.77, Status.BUSY),
            new Employee(104, "赵六", 8, 7777.77, Status.FREE),
            new Employee(104, "赵六", 8, 7777.77, Status.FREE),
            new Employee(105, "田七", 38, 5555.55, Status.BUSY)
);
```
上面的字段分别表示，员工ID,名字，年龄，工资，工作状态

* 获取所有工资的总和
```java
Optional<Double> sum = emps.stream()
            .map(Employee::getSalary)
            .collect(Collectors.reducing(Double::sum));

System.out.println(sum.get());

```

* 将名字转化为字符串，用`,`分隔，收尾加`+`
```java
String str = emps.stream()
            .map(Employee::getName)
            .collect(Collectors.joining("," , "+", "+"));

System.out.println(str);
```

* 分区

工资大于5000一个集合，工资小于5000一个集合
```java
Map<Boolean, List<Employee>> map = emps.stream()
			.collect(Collectors.partitioningBy((e) -> e.getSalary() >= 5000));
		
System.out.println(map);
```

* 分组
1. 单级分组
```java
Map<Status, List<Employee>> map = emps.stream()
                        .collect(Collectors.groupingBy(Employee::getStatus));
            
            System.out.println(map);
```

2. 多级分组
下面就实现了先按照状态分，分类在按照年龄分组。
```java
Map<Status, Map<String, List<Employee>>> map = emps.stream()
			.collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy((e) -> {
				if(e.getAge() >= 60)
					return "老年";
				else if(e.getAge() >= 35)
					return "中年";
				else
					return "成年";
			})));
		
System.out.println(map);

```

* 常用的操作
```java
//获取工资最大
Optional<Double> max = emps.stream()
            .map(Employee::getSalary)
            .collect(Collectors.maxBy(Double::compare));

System.out.println(max.get());

//获取工资最小
Optional<Employee> op = emps.stream()
            .collect(Collectors.minBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));

System.out.println(op.get());

//获取工资的和  返回double
Double sum = emps.stream()
            .collect(Collectors.summingDouble(Employee::getSalary));

System.out.println(sum);

//获取工资平均值 返回double
Double avg = emps.stream()
            .collect(Collectors.averagingDouble(Employee::getSalary));

System.out.println(avg);

//获取总数
Long count = emps.stream()
            .collect(Collectors.counting());

System.out.println(count);

System.out.println("--------------------------------------------");


//通过这种形式也能获取上面的值 看DoubleSummaryStatistics的方法
DoubleSummaryStatistics dss = emps.stream()
            .collect(Collectors.summarizingDouble(Employee::getSalary));
System.out.println(dss.getMax());

```


# 串行流和并行流