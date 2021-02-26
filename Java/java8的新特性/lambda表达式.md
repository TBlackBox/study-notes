# 简介
Lambda是一个匿名的函数，可以吧Lambda表达式理解为`一段可传递的代码`（将代码像数据一样传递）。可以写出更简洁。更灵活的代码。使java的语言表达能力得到了提升。


# 基础语法
## 新的操作符
`->` 箭头操作符或者Lambda操作符
该操作符将lambda分为两部分
- 左侧：参数列表。
- 右侧：抽象方法的实现，即需要执行的功能。

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
() -> System.out.println("嘿嘿");
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

5. lambda的参数列表数据类型可以不写，JVM编译器能通过上下文推断出数据类型，如果要写，整个列表都要写。


# 内置的四大核心函数式接口
这里列出4个核心接口，还有很多接口，都是这4个的子接口。这些接口都可以在`java.util.function`包下能看到

## 消费型接口
```
Consumer<T>
    void accept(T t);
```

例子：
```
public class TestLambad {
    public static void main(String[] args) {
    	new test().consumer(1200,(e) -> System.out.println("吃饭消费："+ e+ "元"));
    }
}
class test{
	 public void consumer(Integer money,Consumer<Integer> con) {
	    	con.accept(money);
	    }
}
```

## 共给型接口
```
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

用于获取一个值（注意理解），

## 函数型接口
```
Function<T, R>
     R apply(T t);
```
穿一个参数，处理后返回一个值

## 断言型接口
```
Predicate<T>
     boolean test(T t);
```
判断接口，通常用于过滤这些。


# 引用

为方便说明；我们创建一个对象user
```
public class User {

	private String name;
	private Integer age;
	
	public User() {
	}
	
	public User(String name, Integer age) {
		super();
		this.name = name;
		this.age = age;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Integer getAge() {
		return age;
	}
	public void setAge(Integer age) {
		this.age = age;
	}
	
}

```
## 方法引用
什么是方法引用啦
若Lambad体中的功能，如果已经有方法提供了实现,可以使用方法引用，可以理解为lambda的表现形式。提供了3种规则,安装这个形式套都可以了。好好理解吧。

## 方法引用的3种规则

1. 对象的引用::实例方法名

```
User user = new User("张三", 25);
//获取名字
Supplier<String> sup = () -> user.getName();
System.out.println("直接获取值："+ sup.get());

//通过方法引用可以这样写,这就是  实例对象::方法名
Supplier<String> sup1 = user::getName;
System.out.println("通过方法引用获取值："+ sup.get());
//直接获取值：张三
//通过方法引用获取值：张三
```

2. 静态类名::静态方法名   

```
Comparator<Integer> com = (x,y) -> Integer.compare(x, y);
System.out.println(com.compare(3, 2));

//静态方法::方法名的形式改写   保证方法的参数跟接口的里面方法的参数一样
Comparator<Integer> com1 = Integer::compare;
System.out.println(com1.compare(3, 2));
```

3. 类名::实例方法名
```
//看两个字符串是否相等
BiPredicate<String, String> bp = (x, y) -> x.equals(y);
System.out.println(bp.test("www", "sss"));

System.out.println("-----------------------------------------");
//类名::方法名的形式改装   需要满足的条件  第一个参数是调用者，第二个参数是方法的参数,第二个参数可以没有
BiPredicate<String, String> bp2 = String::equals;
System.out.println(bp2.test("qw", "wq"));
```

 注意：
 * ①方法引用所引用的方法的参数列表与返回值类型，需要与函数式接口中抽象方法的参数列表和返回值类型保持一致！
 * ②若Lambda 的参数列表的第一个参数，是实例方法的调用者，第二个参数(或无参)是实例方法的参数时，格式： ClassName::MethodName


## 构造器引用
构造器的参数列表，需要与函数式接口中参数列表保持一致！
1. 类名::new
```
Supplier<User> sup = () -> new User();
User user = sup.get();

//改造一哈赛  调用的是无参数够着器 Supplier是不穿参数的，只有返回值
Supplier<User> sup1 = User::new;
User user1 = sup1.get();

//这个就是调用的有2参数的构造器了 BiFunction可以看做是Function的一个子接口
BiFunction<String, Integer, User> user2 = User::new;
```


## 数组引用
类型[]::new

```
//正常的声明一个数组
Function<Integer, String[]> fun = (args) -> new String[args];
String[] strs = fun.apply(10);
System.out.println(strs.length);

System.out.println("--------------------------");

//通过数组引用获取一个用户数组
Function<Integer, User[]> fun2 = User[] :: new;
User[] users = fun2.apply(20);
System.out.println(users.length);
```


# 总结
这里只介绍了基本的用法,这些东西要仔细思考，细细的品味。才能灵活使用。