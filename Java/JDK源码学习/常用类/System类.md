# 简介
System类是一些与系统相关的属性和方法的集合，而且在System类中所有的属性都是静态的，要想引用这些属性和方法，直接使用System类调用即可。

# 常用方法
```
public static void exit(int status)  系统退出 ，如果status为0就表示退出。
public static void gc()   运行垃圾收集机制，调用的是Runtime类中的gc方法。
public static long currentTimeMillis()  返回以毫秒为单位的当前时间。
public static void arraycopy(Object src,int srcPos, Object dest,int desPos,int length) 数组拷贝操作
public static Properties getProperties() 取得当前系统的全部属性。
public static String  getProperty(String key) 根据键值取得属性的具体内容。
```

# 获取当前系统的时间
System类里面有两个回去时间的方式，但他们有区别，并且区别还多大的，先看使用方法。
```java
long timeMill = System.currentTimeMillis();
System.out.println("当前系统毫秒时间：" + timeMill); //当前系统毫秒时间：1608523828707
long timeNano = System.nanoTime();
System.out.println("当前系统纳秒时间：" + timeNano); //当前系统纳秒时间：2945068891755425
```
## 区别
CurrentTimeMillis()
jdk注释说明
```
以毫秒的方式返回当前时间。请注意，虽然返回值的时间单位是毫秒，但是这个值的粒度取决于底层操作系统并且可能粒度更大。例如，许多操作系统是以几十毫秒为粒度测量时间的。
有关于“计算机时间”和协调世界时（UTC）之间的细微差别， 请查阅“Date”类。
@return 当前时间和1970年1月1日午夜之间的差值，以毫秒来测量。
```
### 补充说明
1. 从源码中可以看到，这个方式是一个native方法，该值由底层提供。
2. 该方法可以用来计算当前日期，当前星期几等，与Date的换算非常方便，JDK提供了相关的接口来换算。
3. 通过该方法获取的值的依据是当前系统的日期和时间，可以在系统设置中进行设置和修改。


nanoTime()
```
返回正在运行的Java虚拟机的高分辨率时间源的当前值，以纳秒计。
该方法可能仅仅用于测量已经逝去的时间，并且与任何其它系统或者挂钟时间概念无关。该返回值表示从某个固定但任意的原点时间（可能在未来，所以值可能是负数）开始的纳秒数。在一个java虚拟机实例中，所有该方法的调用都使用相同的原点；其它虚拟机实例很可能使用不同的源头。

该方法提供了纳秒级别的精度，但是不一定是纳秒级分辨率（也就是该值改变的频率）———— 除非这个分辨率至少和currentTimeMillis()一样好，否则将不会做任何保证。

在跨越大于292年（2的63次方纳秒）左右的连续调用中，这个差值将不能正确地计算已经过去的时间，因为数字溢出。

仅仅只有当在同一java虚拟机实例中获取的两个值之间的差值被计算时，返回值才有意义。

例如，去测量某代码执行花费了多长时间：
long startTime = System.nanoTime(); 
//...被测量的代码... 
long estimatedTime = System.nanoTime() - startTime;

要比较两个nanoTime的值：
long t0 = System.nanoTime();

long t1 = System.nanoTime()。
因为数字溢出的可能性，您应该使用"t1 - t0 < 0"，而不是"t1 < t0"（来判断它们的大小，笔者注）。
@return 当前正在运行的java虚拟机的高精度时间资源值，以纳秒为单位。
```

### 补充说明
1. 该方法也是一个本地方法，返回值由底层提供。
2. 如注释中所说，该方法从java 1.5开始引入。 
3. 该方法所基于的时间是随机的，但在同一个JVM中，不同的地方使用的原点时间是一样的。

## 两者的区别与选择
前面对System.currentTimeMillis()和System.nanoTime()都分别从源码注释的角度进行了介绍，算是比较详细了，这里再简单终结一下，顺便谈一谈工作中如何选择：
    
- System.nanoTime()的精确度更高一些，如今的硬件设备性能越来越好，如果要更精密计算执行某行代码或者某块代码所消耗的时间，该方法会测量得更精确。开发者可以根据需要的精确度来选择用哪一个方法。
- 单独获取System.nanoTime()没有什么意义，因为该值是随机的，无法表示当前的时间。如果要记录当前时间点，用System.currentTimeMills()。
- System.currentTimeMills()得到的值能够和Date类方便地转换，jdk提供了多个接口来实现；但是System.nanoTime()则不行。
- System.currentTimeMills()的值是基于系统时间的，可以人为地进行修改；而System.nanoTime()则不能，所以如文章开头笔者碰到的问题一样，如果需要根据时间差来过滤某些频繁的操作，用System.nanoTime()会比较合适。

# 列出系统的所有属性
```java
System.getProperties().list(System.out) ;	// 列出系统的全部属性
```
输出
```
java.runtime.name=Java(TM) SE Runtime Environment
sun.boot.library.path=C:\Program Files\Java\jdk1.8.0_211\jr...
java.vm.version=25.211-b12
java.vm.vendor=Oracle Corporation
java.vendor.url=http://java.oracle.com/
path.separator=;
java.vm.name=Java HotSpot(TM) 64-Bit Server VM
file.encoding.pkg=sun.io
user.script=
user.country=CN
sun.java.launcher=SUN_STANDARD
sun.os.patch.level=Service Pack 1
java.vm.specification.name=Java Virtual Machine Specification
user.dir=D:\Jason\project\code\study-example\s...
java.runtime.version=1.8.0_211-b12
java.awt.graphicsenv=sun.awt.Win32GraphicsEnvironment
java.endorsed.dirs=C:\Program Files\Java\jdk1.8.0_211\jr...
os.arch=amd64
java.io.tmpdir=C:\Users\ADMINI~1.BF-\AppData\Local\T...
line.separator=

java.vm.specification.vendor=Oracle Corporation
user.variant=
os.name=Windows 7
sun.jnu.encoding=GBK
java.library.path=C:\Program Files\Java\jdk1.8.0_211\bi...
java.specification.name=Java Platform API Specification
java.class.version=52.0
sun.management.compiler=HotSpot 64-Bit Tiered Compilers
os.version=6.1
user.home=C:\Users\Administrator.BF-20180801KGCC
user.timezone=
java.awt.printerjob=sun.awt.windows.WPrinterJob
file.encoding=UTF-8
java.specification.version=1.8
user.name=Administrator
java.class.path=D:\Jason\project\code\study-example\s...
java.vm.specification.version=1.8
sun.arch.data.model=64
java.home=C:\Program Files\Java\jdk1.8.0_211\jre
sun.java.command=org.eclipse.jdt.internal.junit.runner...
java.specification.vendor=Oracle Corporation
user.language=zh
awt.toolkit=sun.awt.windows.WToolkit
java.vm.info=mixed mode
java.version=1.8.0_211
java.ext.dirs=C:\Program Files\Java\jdk1.8.0_211\jr...
sun.boot.class.path=C:\Program Files\Java\jdk1.8.0_211\jr...
java.vendor=Oracle Corporation
file.separator=\
java.vendor.url.bug=http://bugreport.sun.com/bugreport/
sun.cpu.endian=little
sun.io.unicode.encoding=UnicodeLittle
sun.desktop=windows
sun.cpu.isalist=amd64
```
### 设置和获取单独属性
```java
//设置系统的
System.setProperty("sun.desktop", "Linux");

String desktop = System.getProperty("sun.desktop");
System.out.println(desktop);//Linux
//自定义
System.setProperty("test", "哈哈");
System.out.println(System.getProperty("test")); //哈哈
```

# 垃圾对象的回收
一个对象如果不使用，则肯定要等待进行垃圾收集，垃圾收集可以自动调用也可以手工调用，手工调用的时候就是调用System.gc()或者Runtime.getRuntime().gc()。但是，如果一个对象在回收之前需要做一些收尾工作，则就必须覆写Object类中的：
protected void finalize() throws Throwable
在对象被回收之前调用，以处理对象回收前的若干操作，例如释放资源等等。
 ```java
 class Person{
	private String name ;
	private int age ;
	public Person(String name,int age){
		this.name = name ;
		this.age = age;
	}
	public String toString(){	// 覆写toString()方法
		return "姓名：" + this.name + "，年龄：" + this.age ;
	}
	public void finalize() throws Throwable{	// 对象释放空间时默认调用此方法
		System.out.println("对象被释放 --> " + this) ;
	}
};

public class SystemDemo04{
	public static void main(String args[]){
		Person per = new Person("张三",30) ;
		per = null ;	// 断开引用
		System.gc() ;		// 强制性释放空间
	}
};

 ```

# System.arraycopy（）方法
把一个数组中某一段字节数据放到另一个数组中。至于从第一个数组中取出几个数据，放到第二个数组中的什么位置都是可以通知这个方法的参数控制的。
```
public static void arraycopy(Object src, int srcPos, Object dest, int ]
destPos, int length)
```
-  src:源数组;
- srcPos:源数组要复制的起始位置;
- dest:目的数组;
- destPos:目的数组放置的起始位置;

- length:复制的长度.

一般的高手，看完上面的定义基本就会用来，不会的话，也可以看看下面的示例。

注意：src 和 dest都必须是同类型或者可以进行转换类型的数组．
例子：
```
int[] src = {1,2,3,4,5};

int[] dest = {10,11,12,13,14};

System.arraycopy(src, 1, dest, 2, 2);

//dest [10,11,2,3,14]
```
### 要求
1. 源的起始位置+长度 不能超过末尾
2. 目标起始位置+长度 不能超过末尾
3. 且所有的参数不能为负数

# 总结
System类本身提供的静态属性都是与IO操作有关的。在IO操作中还会有更多System类的使用，可以使用system类取得计算时间，以及通过gc()方法进行垃圾收集 ,此方法是包装了Runtime类中的gc()方法,arraycopy()方法在复制数组的使用也经常使用，例如在jdk源码中`CopyOnWriteArrayList`的add()方法等。
