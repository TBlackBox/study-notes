## Constructor类及其用法

Constructor类存在于反射包(java.lang.reflect)中，反映的是Class 对象所表示的类的构造方法。获取Constructor对象是通过Class类中的方法获取的，Class类与Constructor相关的主要方法如下：

| 方法返回值      | 方法名称                                          | 方法说明                                                  |
| --------------- | ------------------------------------------------- | --------------------------------------------------------- |
| `static Class`  | forName(String className)                         | 返回与带有给定字符串名的类或接口相关联的 Class 对象。     |
| `Constructor`   | `getConstructor(Class... parameterTypes)`         | 返回指定参数类型、具有public访问权限的构造函数对象        |
| `Constructor[]` | getConstructors()                                 | 返回所有具有public访问权限的构造函数的Constructor对象数组 |
| `Constructor`   | `getDeclaredConstructor(Class... parameterTypes)` | 返回指定参数类型、所有声明的（包括private）构造函数对象   |
| `Constructor[]` | `getDeclaredConstructor()`                        | 返回所有声明的（包括private）构造函数对象                 |
| T               | newInstance()                                     | 创建此 Class 对象所表示的类的一个新实例。                 |



下面看一个简单例子来了解Constructor对象的使用：

```java
package reflect;

import java.io.Serializable;
import java.lang.reflect.Constructor;

public class ReflectDemo implements Serializable{
    public static void main(String[] args) throws Exception {

        Class<?> clazz = null;

        //获取Class对象的引用
        clazz = Class.forName("reflect.User");

        //第一种方法，实例化默认构造方法，User必须无参构造函数,否则将抛异常
        User user = (User) clazz.newInstance();
        user.setAge(20);
        user.setName("Rollen");
        System.out.println(user);

        System.out.println("--------------------------------------------");

        //获取带String参数的public构造函数
        Constructor cs1 =clazz.getConstructor(String.class);
        //创建User
        User user1= (User) cs1.newInstance("xiaolong");
        user1.setAge(22);
        System.out.println("user1:"+user1.toString());

        System.out.println("--------------------------------------------");

        //取得指定带int和String参数构造函数,该方法是私有构造private
        Constructor cs2=clazz.getDeclaredConstructor(int.class,String.class);
        //由于是private必须设置可访问
        cs2.setAccessible(true);
        //创建user对象
        User user2= (User) cs2.newInstance(25,"lidakang");
        System.out.println("user2:"+user2.toString());

        System.out.println("--------------------------------------------");

        //获取所有构造包含private
        Constructor<?> cons[] = clazz.getDeclaredConstructors();
        // 查看每个构造方法需要的参数
        for (int i = 0; i < cons.length; i++) {
            //获取构造函数参数类型
            Class<?> clazzs[] = cons[i].getParameterTypes();
            System.out.println("构造函数["+i+"]:"+cons[i].toString() );
            System.out.print("参数类型["+i+"]:(");
            for (int j = 0; j < clazzs.length; j++) {
                if (j == clazzs.length - 1)
                    System.out.print(clazzs[j].getName());
                else
                    System.out.print(clazzs[j].getName() + ",");
            }
            System.out.println(")");
        }
    }
}


class User {
    private int age;
    private String name;
    public User() {
        super();
    }
    public User(String name) {
        super();
        this.name = name;
    }

    /**
     * 私有构造
     * @param age
     * @param name
     */
    private User(int age, String name) {
        super();
        this.age = age;
        this.name = name;
    }

  //..........省略set 和 get方法
}
```

运行结果：

```java
User [age=20, name=Rollen]
--------------------------------------------
user1:User [age=22, name=xiaolong]
--------------------------------------------
user2:User [age=25, name=lidakang]
--------------------------------------------
构造函数[0]:private reflect.User(int,java.lang.String)
参数类型[0]:(int,java.lang.String)
构造函数[1]:public reflect.User(java.lang.String)
参数类型[1]:(java.lang.String)
构造函数[2]:public reflect.User()
参数类型[2]:()
```

关于Constructor类本身一些常用方法如下(仅部分，其他可查API)，

| 方法返回值 | 方法名称                          | 方法说明                                                     |
| ---------- | --------------------------------- | ------------------------------------------------------------ |
| `Class`    | `getDeclaringClass()`             | 返回 Class 对象，该对象表示声明由此 Constructor 对象表示的构造方法的类,其实就是返回真实类型（不包含参数） |
| `Type[]`   | `getGenericParameterTypes()`      | 按照声明顺序返回一组 Type 对象，返回的就是 Constructor对象构造函数的形参类型。 |
| `String`   | `getName()`                       | 以字符串形式返回此构造方法的名称。                           |
| `Class[]`  | `getParameterTypes()`             | 按照声明顺序返回一组 Class 对象，即返回Constructor 对象所表示构造方法的形参类型 |
| `T`        | `newInstance(Object... initargs)` | 使用此 Constructor对象表示的构造函数来创建新实例             |
| String     | `toGenericString()`               | 返回描述此 Constructor 的字符串，其中包括类型参数。          |



代码演示如下：

```java
Constructor cs3=clazz.getDeclaredConstructor(int.class,String.class);

System.out.println("-----getDeclaringClass-----");
Class uclazz=cs3.getDeclaringClass();
//Constructor对象表示的构造方法的类
System.out.println("构造方法的类:"+uclazz.getName());

System.out.println("-----getGenericParameterTypes-----");
//对象表示此 Constructor 对象所表示的方法的形参类型
Type[] tps=cs3.getGenericParameterTypes();
for (Type tp:tps) {
    System.out.println("参数名称tp:"+tp);
}
System.out.println("-----getParameterTypes-----");
//获取构造函数参数类型
Class<?> clazzs[] = cs3.getParameterTypes();
for (Class claz:clazzs) {
    System.out.println("参数名称:"+claz.getName());
}
System.out.println("-----getName-----");
//以字符串形式返回此构造方法的名称
System.out.println("getName:"+cs3.getName());

System.out.println("-----getoGenericString-----");
//返回描述此 Constructor 的字符串，其中包括类型参数。
System.out.println("getoGenericString():"+cs3.toGenericString());
/**
 输出结果:
 -----getDeclaringClass-----
 构造方法的类:reflect.User
 -----getGenericParameterTypes-----
 参数名称tp:int
 参数名称tp:class java.lang.String
 -----getParameterTypes-----
 参数名称:int
 参数名称:java.lang.String
 -----getName-----
 getName:reflect.User
 -----getoGenericString-----
 getoGenericString():private reflect.User(int,java.lang.String)
 */
```

其中关于Type类型这里简单说明一下，Type 是 Java 编程语言中所有类型的公共高级接口。它们包括原始类型、参数化类型、数组类型、类型变量和基本类型。`getGenericParameterTypes` 与 `getParameterTypes` 都是获取构成函数的参数类型，前者返回的是Type类型，后者返回的是Class类型，由于Type顶级接口，Class也实现了该接口，因此Class类是Type的子类，Type 表示的全部类型而每个Class对象表示一个具体类型的实例，如`String.class`仅代表String类型。由此看来Type与 Class 表示类型几乎是相同的，只不过 Type表示的范围比Class要广得多而已。当然Type还有其他子类，如：

- TypeVariable：表示类型参数，可以有上界，比如：T extends Number
- ParameterizedType：表示参数化的类型，有原始类型和具体的类型参数，比如：`List`
- WildcardType：表示通配符类型，比如：?, ? extends Number, ? super Integer

通过以上的分析，对于Constructor类已有比较清晰的理解，利用好Class类和Constructor类，我们可以在运行时动态创建任意对象，从而突破必须在编译期知道确切类型的障碍。

通过以上的分析，对于Constructor类已有比较清晰的理解，利用好Class类和Constructor类，我们可以在运行时动态创建任意对象，从而突破必须在编译期知道确切类型的障碍。