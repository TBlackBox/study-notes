# 简介
Optional<T>是一个容器类，代表一个值存在或者不存在，原来用null表示一个值得不存在，现在Optional可以更好的表达一个概念，并且可以避免空指针异常。



# 常用方法
1. 创建实例
```
//Optional.of(T t) : 创建一个 Optional 实例   t不能为空
Optional<String> op = Optional.of("哈哈");

//创建一个空的 Optional 实例
Optional<String> op1 = Optional.empty();

//Optional.ofNullable(T t):若 t 不为 null,创建 Optional 实例,否则创建空实例
Optional<String> op2 = Optional.ofNullable("哈哈");
```

2. 判断容器中是否包含了值
```
boolean b = op.isPresent();
```

3. 有值则返回原来值 ，没有值 返回没有定义的值
```
//orElse(T t) :  如果调用对象包含值，返回该值，否则返回t
String op4 = op.orElse("没有值时我返回");

//orElseGet(Supplier s) :如果调用对象包含值，返回该值，否则返回 s 获取的值 Supplier 供给型
String op5 = op.orElseGet(() -> {
            String a = "王麻子";
            return a;
});

```

4. 对值进行处理后返回
```
//map(Function f): 如果有值对其处理，并返回处理后的Optional，否则返回 Optional.empty()
Optional<String> op6 = op.map(String::toLowerCase);

//flatMap(Function mapper):与 map 类似，要求返回值必须是Optional
Optional<String> op7 = op.flatMap((e) -> Optional.of("么么哒"));
```

# 可以用在什么地方
在不知道这个值是否为空的情况下，可以给把这个值定义到Optional容器中，还有就是定义类的属性的是有，不确定的情况先定义它
例如：
用户的的儿子,可能有儿子，可能没有,可以这样定义
```
private Optional<Son>  opt = Optional.empty();
```

# 总结 
这里只写了几个常用的方法，更多的，可以去看文档或者实现类