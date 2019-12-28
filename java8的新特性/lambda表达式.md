# 简介
Lambda是一个匿名的函数，可以吧Lambda表达式理解为`一段可传递的代码`（将代码像数据一样传递）。可以写出更简洁。更灵活的代码。使java的于洋表达能力得到了提升。


# 基础语法
## 新的操作符
`->` 箭头操作符或者Lambda操作符
该操作符将lambda分为两部分
左侧：参数列表。
右侧：抽象方法的实现，即需要执行的功能。

## 支持条件
1. 需要`函数式接口`的支持
函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。函数式接口可以被隐式转换为 lambda 表达式。
例如：
```
@FunctionalInterface
interface GreetingService 
{
    void sayMessage(String message);
}
```
`@FunctionalInterface`可以检查是不是函数式接口

## 几种格式 
1. 无参数,无返回值
```
() ->System.out.println("嘿嘿");
```
例如：
```
Runnable r = new Runnable() {
                        @Override
                        public void run() {
                                    System.out.println("Hello World!" + num);
                        }
            };

r.run();
```

通过lambda实现：
```
Runnable r1 = () -> System.out.println("Hello Lambda!");
r1.run();
```

2. 有一个参数,无返回值
```
(param1) ->System.out.println(param1);
```
例子：
```
Consumer<String> con = x -> System.out.println(x);
		con.accept("我大尚硅谷威武！");
```
注意：如果只有一个参数,()可以不写
```
param1 ->System.out.println(param1);
```
3. 多参数，又返回值，方法体里面有多行
```
Comparator<Integer> com = (x, y) -> {
                        System.out.println("函数式接口");
                        return Integer.compare(x, y);
            };
```

4. lambda只有一条语句,{},return都可以不写
```
Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
```

5. lambda的参数列表数据类型可以不写，JVM编译器能通过上下文推断出数据类型，如果要写，整个列表都要写