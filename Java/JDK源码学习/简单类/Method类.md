## Method类及其用法

Method 提供关于类或接口上单独某个方法（以及如何访问该方法）的信息，所反映的方法可能是类方法或实例方法（包括抽象方法）。下面是Class类获取Method对象相关的方法：

| 方法返回值 | 方法名称                                                  | 方法说明                                                     |
| ---------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| `Method`   | `getDeclaredMethod(String name, Class... parameterTypes)` | 返回一个指定参数的Method对象，该对象反映此 Class 对象所表示的类或接口的指定已声明方法。 |
| `Method[]` | `getDeclaredMethod()`                                     | 返回 Method 对象的一个数组，这些对象反映此 Class 对象表示的类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。 |
| `Method`   | `getMethod(String name, Class... parameterTypes)`         | 返回一个 Method 对象，它反映此 Class 对象所表示的类或接口的指定公共成员方法。 |
| `Method[]` | `getMethods()`                                            | 返回一个包含某些 Method 对象的数组，这些对象反映此 Class 对象所表示的类或接口（包括那些由该类或接口声明的以及从超类和超接口继承的那些的类或接口）的公共 member 方法。 |



同样通过案例演示上述方法：

```java
import java.lang.reflect.Method;

public class ReflectMethod  {


    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException {

        Class clazz = Class.forName("reflect.Circle");

        //根据参数获取public的Method,包含继承自父类的方法
        Method method = clazz.getMethod("draw",int.class,String.class);

        System.out.println("method:"+method);

        //获取所有public的方法:
        Method[] methods =clazz.getMethods();
        for (Method m:methods){
            System.out.println("m::"+m);
        }

        System.out.println("=========================================");

        //获取当前类的方法包含private,该方法无法获取继承自父类的method
        Method method1 = clazz.getDeclaredMethod("drawCircle");
        System.out.println("method1::"+method1);
        //获取当前类的所有方法包含private,该方法无法获取继承自父类的method
        Method[] methods1=clazz.getDeclaredMethods();
        for (Method m:methods1){
            System.out.println("m1::"+m);
        }
    }

/**
     输出结果:
     method:public void reflect.Shape.draw(int,java.lang.String)

     m::public int reflect.Circle.getAllCount()
     m::public void reflect.Shape.draw()
     m::public void reflect.Shape.draw(int,java.lang.String)
     m::public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
     m::public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
     m::public final void java.lang.Object.wait() throws java.lang.InterruptedException
     m::public boolean java.lang.Object.equals(java.lang.Object)
     m::public java.lang.String java.lang.Object.toString()
     m::public native int java.lang.Object.hashCode()
     m::public final native java.lang.Class java.lang.Object.getClass()
     m::public final native void java.lang.Object.notify()
     m::public final native void java.lang.Object.notifyAll()

     =========================================
     method1::private void reflect.Circle.drawCircle()

     m1::public int reflect.Circle.getAllCount()
     m1::private void reflect.Circle.drawCircle()
     */
}

class Shape {
    public void draw(){
        System.out.println("draw");
    }

    public void draw(int count , String name){
        System.out.println("draw "+ name +",count="+count);
    }

}
class Circle extends Shape{

    private void drawCircle(){
        System.out.println("drawCircle");
    }
    public int getAllCount(){
        return 100;
    }
}
```

在通过getMethods方法获取Method对象时，会把父类的方法也获取到，如上的输出结果，把Object类的方法都打印出来了。而getDeclaredMethod/getDeclaredMethods方法都只能获取当前类的方法。我们在使用时根据情况选择即可。下面将演示通过Method对象调用指定类的方法：

```java
Class clazz = Class.forName("reflect.Circle");
//创建对象
Circle circle = (Circle) clazz.newInstance();

//获取指定参数的方法对象Method
Method method = clazz.getMethod("draw",int.class,String.class);

//通过Method对象的invoke(Object obj,Object... args)方法调用
method.invoke(circle,15,"圈圈");

//对私有无参方法的操作
Method method1 = clazz.getDeclaredMethod("drawCircle");
//修改私有方法的访问标识
method1.setAccessible(true);
method1.invoke(circle);

//对有返回值得方法操作
Method method2 =clazz.getDeclaredMethod("getAllCount");
Integer count = (Integer) method2.invoke(circle);
System.out.println("count:"+count);

/**
    输出结果:
    draw 圈圈,count=15
    drawCircle
    count:100
*/
```

在上述代码中调用方法，使用了Method类的`invoke(Object obj,Object... args)`第一个参数代表调用的对象，第二个参数传递的调用方法的参数。这样就完成了类方法的动态调用。

| 方法返回值 | 方法名称                             | 方法说明                                                     |
| ---------- | ------------------------------------ | ------------------------------------------------------------ |
| `Object`   | `invoke(Object obj, Object... args)` | 对带有指定参数的指定对象调用由此 Method 对象表示的底层方法。 |
| `Class`    | `getReturnType()`                    | 返回一个 Class 对象，该对象描述了此 Method 对象所表示的方法的正式返回类型,即方法的返回类型 |
| Type       | `getGenericReturnType()`             | 返回表示由此 Method 对象所表示方法的正式返回类型的 Type 对象，也是方法的返回类型。 |
| `Class[]`  | `getParameterTypes()`                | 按照声明顺序返回 Class 对象的数组，这些对象描述了此 Method 对象所表示的方法的形参类型。即返回方法的参数类型组成的数组 |
| `Type[]`   | `getGenericParameterTypes()`         | 按照声明顺序返回 Type 对象的数组，这些对象描述了此 Method 对象所表示的方法的形参类型的，也是返回方法的参数类型 |
| `String`   | `getName()`                          | 以 String 形式返回此 Method 对象表示的方法名称，即返回方法的名称 |
| `boolean`  | `isVarArgs()`                        | 判断方法是否带可变参数，如果将此方法声明为带有可变数量的参数，则返回 true；否则，返回 false。 |
| `String`   | `toGenericString()`                  | 返回描述此 Method 的字符串，包括类型参数。                   |



 getReturnType方法/getGenericReturnType方法都是获取Method对象表示的方法的返回类型，只不过前者返回的Class类型后者返回的Type(Type类的文章有)，Type就是一个接口而已，在Java8中新增一个默认的方法实现，返回的就参数类型信息

```java
public interface Type {
    //1.8新增
    default String getTypeName() {
        return toString();
    }
}
```

而getParameterTypes/getGenericParameterTypes也是同样的道理，都是获取Method对象所表示的方法的参数类型，其他方法与前面的Field和Constructor是类似的。