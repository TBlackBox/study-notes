# 简介
Optional<T>是一个容器类，代表一个值存在或者不存在，原来用null表示一个值得不存在，现在Optional可以更好的表达一个概念，并且可以避免空指针异常。



# 常用方法
1.of:为非null的值创建一个Optional。of方法通过工厂方法创建Optional类。需要注意的是，创建对象时传入的参数不能为null。如果传入参数为null，则抛出NullPointerException 。

```java
Optional<String> optional = Optional.of("xiaoming");//传入null，抛出NullPointerExceptionOptional<Object> o = Optional.of(null);
```

2.ofNullable:为指定的值创建一个Optional，如果指定的值为null，则返回一个空的Optional。

```
//创建一个空的 Optional 实例
Optional<String> op1 = Optional.empty();

//Optional.ofNullable(T t):若 t 不为 null,创建 Optional 实例,否则创建空实例
Optional<String> op2 = Optional.ofNullable("哈哈");
```

3.isPresent：值存在返回true，否则返回false

```
Optional<String> optiona2 = Optional.of("xiaoming");System.out.println(optiona2.isPresent());
```

4.get:Optional有值就返回，没有抛出NoSuchElementException

```
Optional<Object> o1 = Optional.ofNullable(null);
System.out.println(o1.get());
```

5.ifPresent:如果Optional有值则调用consumer处理，否则不处理

```
Optional<Object> o1 = Optional.ofNullable(null); 
o1.ifPresent(s -> System.out.println(s));
```

6.orElse：如果有值则将其返回，否则返回指定的其它值

```
Optional<Object> o1 = Optional.ofNullable(null);
System.out.println(o1.orElse("输出orElse")); // 输出orElse
```

7.orElseGet：orElseGet与orElse方法类似，区别在于得到的默认值。orElse方法将传入的字符串作为默认值，orElseGet方法可以接受Supplier接口的实现用来生成默认值

```
Optional<Object> o1 = Optional.ofNullable(null);
System.out.println(o1.orElseGet(() -> "default value")); // default value
```

------

*注意：orElse 和 orElseGet 看似差不多，其实有很大不同；看下面例子

```java
Shop shop = null;
System.out.println("test orElse");
Optional.ofNullable(shop).orElse(createNew());
System.out.println("test orElseGet");
Optional.ofNullable(shop).orElseGet(()->createNew());
//createNew
private static Shop createNew() {  System.out.println("create new");
  return new Shop("丝袜", 50);
}//输出：
test orElsecreate newtest orElseGetcreate new
Shop shop = new Shop("长腿丝袜",100);
System.out.println("test orElse");
Optional.ofNullable(shop).orElse(createNew());System.out.println("test orElseGet");
Optional.ofNullable(shop).orElseGet(()->createNew());
//输出
test orElsecreate newtest orElseGet
```

从上面两个例子可以看到，当Optional 为空时，orElse和orElseGet 区别不大，但当Optional有值时，orElse仍然会去调用方法创建对象，而orElseGet不会再调用方法；在我们处理的业务数据量大的时候，这两者的性能就有很大的差异。

------

8.orElseThrow：如果有值则将其返回，否则抛出supplier接口创建的异常。

```java
Optional<Object> o1 = Optional.ofNullable(null);
try {  
	o1.orElseThrow(() -> new Exception("异常！"));
} catch (Exception e) {  
	System.out.println("info:" + e.getMessage());
}//输出：info:异常!
```

``

9.map：如果有值，则对其执行调用mapping函数得到返回值。如果返回值不为null，则创建包含mapping返回值的Optional作为map方法返回值，否则返回空Optional。

```java
Optional<String> optional = Optional.of("xiaoming");
String s = optional.map(e -> e.toUpperCase()).orElse("shiyilingfeng");
System.out.println(s); //输出: XIAOMING
```

10.flatMap：如果有值，为其执行mapping函数返回Optional类型返回值，否则返回空Optional。与map不同的是，flatMap 的返回值必须是Optional

```java
Optional<String> optional = Optional.of("xiaoming");
Optional<String> s = optional.flatMap(e -> Optional.of(e.toUpperCase()));
System.out.println(s.get()); //输出：XIAOMING
```

``

11.filter


```java
List<String> strings = Arrays.asList("rmb", "doller", "ou");
for (String s : strings) { 
    Optional<String> o = Optional.of(s).filter(s1 -> !s1.contains("o"));  		     	 System.out.println(o.orElse("没有不包含o的"));
}//输出：rmb没有不包含o的没有不包含o的
```



# 可以用在什么地方

在不知道这个值是否为空的情况下，可以给把这个值定义到Optional容器中，还有就是定义类的属性的是有，不确定的情况先定义它
例如：
用户的的儿子,可能有儿子，可能没有,可以这样定义

```
private Optional<Son>  opt = Optional.empty();
```

# 总结 
Optional 是java非常有用的一个补充，它旨在减少代码中的NullPointerExceptions，虽然不能百分之百的消除，但也是精心设计的。使用Optional 能更好的帮助我们创建可读性强，bug更少的应用程序。这里只写了几个常用的方法，更多的，可以去看文档或者实现类