[参考文档](https://juejin.im/entry/5bac3ae1e51d450e6b0de9ff)

# RRTI的概念以及Class对象作用

## RTTI（Run-Time Type Identification）运行时类型识别
这个词是C++ 中的概念，Java是《Thinking in Java》一书出来的，其作用是在运行时识别一个对象的类型和类的信息，这里分两种：
- 传统的”RRTI”,它假定我们在编译期已知道了所有类型，如new对象时该类必须已定义好(没有反射哦)。
- 一种是反射机制，它允许我们在运行时发现和使用类型的信息。

*** java中用来表示类型信息的对应类就是Class类，Class类也是一个实实在在的类，存在于JDK的java.lang包中 **


## Class对象作用
Class类被创建后的对象就是Class对象，注意，Class对象表示的是自己手动编写类的类型信息，比如创建一个User类，那么，JVM就会创建一个User对应Class类的Class对象，该Class对象保存了User类相关的类型信息。
实际上在Java中每个类都有一个Class对象，每当我们编写并且编译一个新创建的类就会产生一个对应Class对象并且这个Class对象会被保存在同名.class文件里(编译后的字节码文件保存的就是Class对象)。

### 为什么需要这样一个Class对象呢？
当我们new一个新对象或者引用静态成员变量时，Java虚拟机(JVM)中的类加载器子系统会将对应Class对象加载到JVM中，然后JVM再根据这个类型信息相关的Class对象创建我们需要实例对象或者提供静态变量的引用值。需要特别注意的是，手动编写的每个class类，无论创建多少个实例对象，在JVM中都只有一个Class对象，即在内存中每个类有且只有一个相对应的Class对象。

## class对象的几点总结

- Class类也是类的一种，与class关键字是不一样的。
- 手动编写的类被编译后会产生一个Class对象，其表示的是创建的类的类型信息，而且这个Class对象保存在同名.class的文件中(字节码文件)，比如创建一个User类，编译Shapes类后就会创建其包含User类相关类型信息的Class对象，并保存在User.class字节码文件中。
- 每个通过关键字class标识的类，在内存中有且只有一个与之对应的Class对象来描述其类型信息，无论创建多少个实例对象，其依据的都是用一个Class对象。
- Class类只存私有构造函数，因此对应Class对象只能由JVM创建和加载。
- Class类的对象作用是运行时提供或获得某个对象的类型信息，这点对于反射技术很重要(关于反射稍后分析)。

## Class对象的加载
Class对象是由JVM加载的，那么其加载时机是？实际上所有的类都是在对其第一次使用时动态加载到JVM中的，当程序创建第一个对类的静态成员引用时，就会加载这个被使用的类(实际上加载的就是这个类的字节码文件)，注意，使用new操作符创建类的新实例对象也会被当作对类的静态成员的引用(构造函数也是类的静态方法)，由此看来Java程序在它们开始运行之前并非被完全加载到内存的，其各个部分是按需加载，所以在使用该类时，类加载器首先会检查这个类的Class对象是否已被加载(类的实例对象创建时依据Class对象中类型信息完成的)，如果还没有加载，默认的类加载器就会先根据类名查找.class文件(编译后Class对象被保存在同名的.class文件中)，在这个类的字节码文件被加载时，它们必须接受相关验证，以确保其没有被破坏并且不包含不良Java代码(这是java的安全机制检测)，完全没有问题后就会被动态加载到内存中，此时相当于Class对象也就被载入内存了(毕竟.class字节码文件保存的就是Class对象)，同时也就可以被用来创建这个类的所有实例对象。

## 类的加载过程
```
加载（Loading）--->验证（Verfication）--->准备（Preparation）--->解析（Resolution）--->初始化（Initialization）
```
*** 注意：验证，准备，解析三个过程称为链接（Link）***


- 加载：类加载过程的一个阶段：通过一个类的完全限定查找此类字节码文件，并利用字节码文件创建一个Class对象
- 链接：验证字节码的安全性和完整性，准备阶段正式为静态域分配存储空间，注意此时只是分配静态成员变量的存储空间，不包含实例成员变量，如果必要的话，解析这个类创建的对其他类的所有引用。
- 初始化：类加载最后阶段，若该类具有超类，则对其进行初始化，执行静态初始化器和静态初始化成员变量。

## 获取Class对象的引用
想获取一个类的运行时类型信息并加以使用时，可以通过下面3总方法获取Class对象的引用，这样做的好处是无需通过持有该类的实例对象引用而去获取Class对象。
### 通过`Class.forName`获取
```
Class clazz = Class.forName("test.User");
```
注意调用forName方法时需要捕获一个名称为ClassNotFoundException的异常，因为forName方法在编译器是无法检测到其传递的字符串对应的类是否存在的，只能在程序运行时进行检查，如果不存在就会抛出ClassNotFoundException异常。

### 通过getClass()获取
```
User user = new User();
Class clazz = user.getClass();
```

### 通过Class字面常量
```
Class clazz = User.class;
```
这种方式相对前面两种方法更加简单，更安全。因为它在编译器就会受到编译器的检查同时由于无需调用forName方法效率也会更高，因为通过字面量的方法获取Class对象的引用不会自动初始化该类(即上面的类加载过程中的初始化不会自动执行)。更加有趣的是字面常量的获取Class对象引用方式不仅可以应用于普通的类，也可以应用用接口，数组以及基本数据类型，这点在反射技术应用传递参数时很有帮助。
由于基本数据类型还有对应的基本包装类型，其包装类型有一个标准字段TYPE，而这个TYPE就是一个引用，指向基本数据类型的Class对象，其等价转换如下，一般情况下更倾向使用.class的形式，这样可以保持与普通类的形式统一。
```
boolean.class = Boolean.TYPE;
char.class = Character.TYPE;
byte.class = Byte.TYPE;
short.class = Short.TYPE;
int.class = Integer.TYPE;
long.class = Long.TYPE;
float.class = Float.TYPE;
double.class = Double.TYPE;
void.class = Void.TYPE;
```

### 结论

- 获取Class对象引用的方式3种，通过继承自Object类的getClass方法，Class类的静态方法forName以及字面常量的方式”.class”。
- 其中实例类的getClass方法和Class类的静态方法forName都将会触发类的初始化阶段，而字面常量获取Class对象的方式则不会触发初始化。
- 初始化是类加载的最后一个阶段，也就是说完成这个阶段后类也就加载到内存中(Class对象在加载阶段已被创建)，此时可以对类进行各种必要的操作了（如new对象，调用静态成员等），注意在这个阶段，才真正开始执行类中定义的Java程序代码或者字节码。

关于类加载的初始化阶段，在虚拟机规范严格规定了有且只有5种场景必须对类进行初始化：

- 使用new关键字实例化对象时、读取或者设置一个类的静态字段(不包含编译期常量)以及调用静态方法的时候，必须触发类加载的初始化过程(类加载过程最终阶段)。
- 使用反射包(java.lang.reflect)的方法对类进行反射调用时，如果类还没有被初始化，则需先进行初始化，这点对反射很重要。
- 当初始化一个类的时候，如果其父类还没进行初始化则需先触发其父类的初始化。
- 当Java虚拟机启动时，用户需要指定一个要执行的主类(包含main方法的类)，虚拟机会先初始化这个主类
- 当使用JDK 1.7 的动态语言支持时，如果一个java.lang.invoke.MethodHandle 实例最后解析结果为REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄对应类没有初始化时，必须触发其初始化(这点看不懂就算了，这是1.7的新增的动态语言支持，其关键特征是它的类型检查的主体过程是在运行期而不是编译期进行的，这是一个比较大点的话题，这里暂且打住)





### 泛化的Class对象引用

由于Class的引用总数指向某个类的Class对象，利用Class对象可以创建实例类，这也就足以说明Class对象的引用指向的对象确切的类型。在Java SE5引入泛型后，使用我们可以利用泛型来表示Class对象更具体的类型，即使在运行期间会被擦除，但编译期足以确保我们使用正确的对象类型。如下：

```
//没有泛型
Class intClass = int.class;

//带泛型的Class对象
Class<Integer> integerClass = int.class;

integerClass = Integer.class;

//没有泛型的约束,可以随意赋值
intClass= double.class;

//编译期错误,无法编译通过
//integerClass = double.class
```

从代码可以看出，声明普通的Class对象，在编译器并不会检查Class对象的确切类型是否符合要求，如果存在错误只有在运行时才得以暴露出来。但是通过泛型声明指明类型的Class对象，编译器在编译期将对带泛型的类进行额外的类型检查，确保在编译期就能保证类型的正确性，实际上`Integer.class`就是一个`Class`类的对象。面对下述语句，确实可能令人困惑，但该语句确实是无法编译通过的。

```
//编译无法通过
Class<Number> numberClass=Integer.class;
```

我们或许会想Integer不就是Number的子类吗？然而事实并非这般简单，毕竟Integer的Class对象并非Number的Class对象的子类，前面提到过，所有的Class对象都只来源于Class类，看来事实确实如此。当然我们可以利用通配符“?”来解决问题：

```
Class<?> intClass = int.class;
intClass = double.class;
```

这样的语句并没有什么问题，毕竟通配符指明所有类型都适用，那么为什么不直接使用Class还要使用`Class`呢？这样做的好处是告诉编译器，我们是确实是采用任意类型的泛型，而非忘记使用泛型约束，因此`Class`总是优于直接使用Class，至少前者在编译器检查时不会产生警告信息。当然我们还可以使用extends关键字告诉编译器接收某个类型的子类，如解决前面Number与Integer的问题：

```
//编译通过！
Class<? extends Number> clazz = Integer.class;
//赋予其他类型
clazz = double.class;
clazz = Number.class;
```

上述的代码是行得通的，extends关键字的作用是告诉编译器，只要是Number的子类都可以赋值。这点与前面直接使用`Class`是不一样的。实际上，应该时刻记住向Class引用添加泛型约束仅仅是为了提供编译期类型的检查从而避免将错误延续到运行时期。

### 关于类型转换的问题

在许多需要强制类型转换的场景，我们更多的做法是直接强制转换类型：

```
Animal animal= new Dog();
//强制转换
Dog dog = (Dog) animal;
```
```
interface Animal{ }
class Dog implements  Animal{ }
```

之所可以强制转换，这得归功于RRTI，要知道在Java中，所有类型转换都是在运行时进行正确性检查的，利用RRTI进行判断类型是否正确从而确保强制转换的完成，如果类型转换失败，将会抛出类型转换异常。除了强制转换外，在Java SE5中新增一种使用Class对象进行类型转换的方式，如下：

```
Animal animal= new Dog();
//这两句等同于
Dog dog = (Dog) animal;
Class<Dog> dogType = Dog.class;
Dog dog = dogType.cast(animal)
```

利用Class对象的cast方法，其参数接收一个参数对象并将其转换为Class引用的类型。这种方式似乎比之前的强制转换更麻烦些，确实如此，而且当类型不能正确转换时，仍然会抛出ClassCastException异常。源码如下：

```
public T cast(Object obj) {
    if (obj != null && !isInstance(obj))
         throw new ClassCastException(cannotCastMsg(obj));
     return (T) obj;
  }
```

### instanceof 关键字与isInstance方法

关于instanceof 关键字，它返回一个boolean类型的值，意在告诉我们对象是不是某个特定的类型实例。如下，在强制转换前利用instanceof检测obj是不是Animal类型的实例对象，如果返回true再进行类型转换，这样可以避免抛出类型转换的异常(ClassCastException)

```
public void cast2(Object obj){
    if(obj instanceof Animal){
          Animal animal= (Animal) obj;
      }
}
```

而isInstance方法则是Class类中的一个Native方法，也是用于判断对象类型的，看个简单例子：

```
public void cast2(Object obj){
        //instanceof关键字
        if(obj instanceof Animal){
            Animal animal= (Animal) obj;
        }

        //isInstance方法
        if(Animal.class.isInstance(obj)){
            Animal animal= (Animal) obj;
        }
  }
```

事实上instanceOf 与isInstance方法产生的结果是相同的。对于instanceOf是关键字只被用于对象引用变量，检查左边对象是不是右边类或接口的实例化。如果被测对象是null值，则测试结果总是false。一般形式：

```
//判断这个对象是不是这种类型
obj.instanceof(class)
```

而isInstance方法则是Class类的Native方法，其中obj是被测试的对象或者变量，如果obj是调用这个方法的class或接口的实例，则返回true。如果被检测的对象是null或者基本类型，那么返回值是false;一般形式如下：

```
//判断这个对象能不能被转化为这个类
class.inInstance(obj)
```

最后这里给出一个简单实例，验证isInstance方法与instanceof等价性：

```
class A {}

class B extends A {}

public class C {
  static void test(Object x) {
    print("Testing x of type " + x.getClass());
    print("x instanceof A " + (x instanceof A));
    print("x instanceof B "+ (x instanceof B));
    print("A.isInstance(x) "+ A.class.isInstance(x));
    print("B.isInstance(x) " +
      B.class.isInstance(x));
    print("x.getClass() == A.class " +
      (x.getClass() == A.class));
    print("x.getClass() == B.class " +
      (x.getClass() == B.class));
    print("x.getClass().equals(A.class)) "+
      (x.getClass().equals(A.class)));
    print("x.getClass().equals(B.class)) " +
      (x.getClass().equals(B.class)));
  }
  public static void main(String[] args) {
    test(new A());
    test(new B());
  } 
}
```

执行结果：

```
Testing x of type class com.zejian.A
x instanceof A true
x instanceof B false //父类不一定是子类的某个类型
A.isInstance(x) true
B.isInstance(x) false
x.getClass() == A.class true
x.getClass() == B.class false
x.getClass().equals(A.class)) true
x.getClass().equals(B.class)) false
---------------------------------------------
Testing x of type class com.zejian.B
x instanceof A true
x instanceof B true
A.isInstance(x) true
B.isInstance(x) true
x.getClass() == A.class false
x.getClass() == B.class true
x.getClass().equals(A.class)) false
x.getClass().equals(B.class)) true
```

到此关于Class对象相关的知识点都分析完了，下面将结合Class对象的知识点分析反射技术。

## 常用的一些方法
```
/** 
  *    修饰符、父类、实现的接口、注解相关 
  */

//获取修饰符，返回值可通过Modifier类进行解读
public native int getModifiers();
//获取父类，如果为Object，父类为null
public native Class<? super T> getSuperclass();
//对于类，为自己声明实现的所有接口，对于接口，为直接扩展的接口，不包括通过父类间接继承来的
public native Class<?>[] getInterfaces();
//自己声明的注解
public Annotation[] getDeclaredAnnotations();
//所有的注解，包括继承得到的
public Annotation[] getAnnotations();
//获取或检查指定类型的注解，包括继承得到的
public <A extends Annotation> A getAnnotation(Class<A> annotationClass);
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass);

/** 
  *   内部类相关
  */
//获取所有的public的内部类和接口，包括从父类继承得到的
public Class<?>[] getClasses();
//获取自己声明的所有的内部类和接口
public Class<?>[] getDeclaredClasses();
//如果当前Class为内部类，获取声明该类的最外部的Class对象
public Class<?> getDeclaringClass();
//如果当前Class为内部类，获取直接包含该类的类
public Class<?> getEnclosingClass();
//如果当前Class为本地类或匿名内部类，返回包含它的方法
public Method getEnclosingMethod();

/** 
  *    Class对象类型判断相关
  */
//是否是数组
public native boolean isArray();  
//是否是基本类型
public native boolean isPrimitive();
//是否是接口
public native boolean isInterface();
//是否是枚举
public boolean isEnum();
//是否是注解
public boolean isAnnotation();
//是否是匿名内部类
public boolean isAnonymousClass();
//是否是成员类
public boolean isMemberClass();
//是否是本地类
public boolean isLocalClass(); 

```