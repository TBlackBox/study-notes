https://blog.csdn.net/lililuni/article/details/83449088#1_4

https://www.jianshu.com/p/9be58ee20dee

Java反射
# 定义
反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性，这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。一直以来反射技术都是Java中的闪亮点，这也是目前大部分框架(如Spring/Mybatis等)得以实现的支柱。在Java中，Class类与java.lang.reflect类库一起对反射技术进行了全力的支持。在反射包中，我们常用的类主要有
- Constructor类表示的是Class 对象所表示的类的构造方法，利用它可以在运行时动态创建对象。
- Field表示Class对象所表示的类的成员变量，通过它可以在运行时动态修改成员变量的属性值(包含private)
- Method表示Class对象所表示的类的成员方法，通过它可以动态调用对象的方法(包含private)
# 常用用途
一般第三方应用开发过程中，经常会遇到某个类的某个成员变量、方法或是属性是私有的或是只对系统应用开放，这时候就可以利用Java的反射机制通过反射来获取所需的私有成员或是方法。 如果应用在返回时做了权限验证，对于没有权限的应用返回值是没有意义的缺省值，否则返回实际值起到保护用户的隐私目的。

# 与反射相关的几个类

## Class类

1. Class<T>类在包java.lang包下，有一下几个特点：

+ Class 类的实例对象表示正在运行的 Java 应用程序中的类和接口。也就是jvm中有很多的实例，每个类都有唯一的Class对象。
+ Class 类没有公共构造方法。class 对象实在加载时有java虚拟机以及通过调用类加载器中的defineClass方法自动构建的。
+ Class 对象用于提供类本身的信息，比如有几种构造方法， 有多少属性，有哪些普通方法

2. 获取类对象的几种方式

```
Class.forName（）//（常用）

Hero.class

new Hero().getClass()
```
在同一个类加载器，一个jvm下，一种类，只会有一个类对象存在，所以上面三种方式取出来的对象都一样。
下面通过例子解释：
为了方便，假设我么有一个User类
```
package test;
public class User {

	public String username;		//用户名
	public int sex;				//性别
	public float balance;		//余额
	
	public void setUsername(String username) {
		this.username = username;
	}
}

```

```
package test;
public class Main {
	public static void main(String[] args) {
		
			try {
				//类名(全名)
				String className = "test.User";
	        	//获取类对象   常用
	            Class pClass1 = Class.forName(className);
	            //获取类对象  需要导入类的包，依赖太强，不导包就抛编译错误。
	            Class pClass2 = User.class;
	            //获取类对象  有类了，获取了类对象也没有什么用了（可能特殊情况下可以用）
	            Class pClass3 = new User().getClass();
	            System.out.println(pClass1==pClass2);//输出true
	            System.out.println(pClass1==pClass3);//输出true
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	        }
	}

}

```

3. 该对象有以下常用的方法

|方法|	用途|
|-----|-----|
|asSubclass(Class<U> clazz)|	把传递的类的对象转换成代表其子类的对象|
|Cast	|把对象转换成代表类或是接口的对象|
|getClassLoader()	|获得类的加载器|
|getClasses()	|返回一个数组，数组中包含该类中所有公共类和接口类的对象|
|getDeclaredClasses()	|返回一个数组，数组中包含该类中所有类和接口类的对象|
|forName(String className)	|根据类名返回类的对象|
|getName()	|获得类的完整路径名字|
|newInstance()	|创建类的实例|
|getPackage()	|获得类的包|
|getSimpleName()	|获得类的名字|
|getSuperclass()	|获得当前类继承的父类的名字|
|getInterfaces()	|获得当前类实现的类或是接口|

4. 利用反射机制创建对象
```
1.获取类对象 
Class class = Class.forName("test.User");
2.获取构造器对象 
Constructor constructor = clazz.getConstructor(形参.class);
3 获取对象 
User user =constructor.newInstance(实参);

```

下面有4种获取构造器相关的方法
|方法	|用途|
|------|--------|
|getConstructor(Class...<?> parameterTypes)	|获得该类中与参数类型匹配的公有构造方法|
|getConstructors()	|获得该类的所有公有构造方法|
|getDeclaredConstructor(Class...<?> parameterTypes)	|获得该类中与参数类型匹配的构造方法(可以是私有的，或受保护、默认、公有；)|
|getDeclaredConstructors()	|获得该类所有构造方法（包括私有、受保护、默认、公有）|

接下来都是通过constructor获取对象的实例了，多不展示代码了，

**说明：形参。class,例如：float.class。实参，表示对应的要初始化的数据，例如：5.2f** 

**Constructor 类,代表类的构造方法, 里面有方法  newInstance(Object... initargs)	根据传递的参数创建类的对象**

5. 获取创建对象的属性的方法 

|方法	|用途|
|---------|---------|
|getField(String name)	|获得某个公有的属性对象 |
|getFields()	|获得所有公有的属性对象|
|getDeclaredField(String name)	|获得某个属性对象|
|getDeclaredFields()	|获得所有属性对象|

getField和getDeclaredField的区别
   + getField 只能获取public的，包括从父类继承来的字段。
   + getDeclaredField 可以获取本类所有的字段，包括private的，但是 不能获取继承来的字段。 (注： 这里只能获取到private的字段，但并不能访问该private字段的值,除非加上setAccessible(true))

例如：
```
//类名(全名)
String className = "test.User";
//获取类对象   常用
Class pClass = Class.forName(className);
Constructor Constructor = pClass.getConstructor();

User user =(User) Constructor.newInstance(null);
user.username = "测试";
System.out.println(user.username);//输出 测试

//通过反射修改属性
Field  field = pClass.getDeclaredField("username");
field.set(user, "测试反射");
//field.setAccessible(true);//获取父类的私有属性是，需要加
System.out.println(user.username);//输出  测试反射
//Field 的一些方法
System.out.println(field.get(user));	//测试反射

```
## Field类介绍
Field代表类的成员变量（成员变量也称为类的属性）。  

|方法	|用途|
|------|-----|
|equals(Object obj)	|属性与obj相等则返回true|
|get(Object obj)	|获得obj中对应的属性值|
|set(Object obj, Object value)	|设置obj中对应属性值|

6. 获得类中注解相关的方法

|方法	|用途|
|--------|-------|
|getAnnotation(Class<A> annotationClass)	|返回该类中与参数类型匹配的公有注解对象|
|getAnnotations()	|返回该类所有的公有注解对象|
|getDeclaredAnnotation(Class<A> annotationClass)	|返回该类中与参数类型匹配的所有注解对象|
|getDeclaredAnnotations()	|返回该类所有的注解对象|

上面列举了获取注解的相关方法，由于篇幅原因都不一一介绍了

7。 使用类中的方法 

+ 获取类中方法相关的方法

|方法	|用途|
|-------|------|
|getMethod(String name, Class...<?> parameterTypes)	|获得该类某个公有的方法|
|getMethods()	|获得该类所有公有的方法|
|getDeclaredMethod(String name, Class...<?> parameterTypes)	|获得该类某个方法|
|getDeclaredMethods()	|获得该类所有方法|

**解释**

获取成员方法：
public Method getMethod(String name ，Class<?>… parameterTypes):获取"公有方法"；（包含了父类的方法也包含Object类）
public Method getDeclaredMethods(String name ，Class<?>… parameterTypes) :获取成员方法，包括私有的(不包括继承的)
参数解释：
  name : 方法名；
  Class … : 形参的Class类型对象
调用方法  
```
Method --> public Object invoke(Object obj,Object… args):  
```
参数说明：

  - obj : 要调用方法的对象； 

-   args:调用方式时所传递的实参；

调用实例

```
	//获取类对象   常用
	Class pClass = Class.forName(className);
	Constructor Constructor = pClass.getConstructor();
	User user =(User) Constructor.newInstance(null);
	
	Method m = pClass.getMethod("setUsername", String.class);
	m.invoke(user, "haha");
	//通过反射修改属性
	Field  field = pClass.getDeclaredField("username");
	System.out.println(field.get(user));

```

# 通过反射获取枚举的值

```
//获取枚举类对象 可以通过上面 介绍的3种方式获取
Class classEnum = Class.forName(typeOfT.getTypeName());
//获取枚举内容  Enum<?> 可以转换为特定的类型
Enum<?>[] enums = (Enum<?>[]) classEnum.getEnumConstants();
```


# 总结

+ 理解反射后，你会发现发射很强大。特别是在需要切换业务时，假如通过非反射的方法实现，需要修改代码，new 不同的对象。但如果使用反射，就只需要修改配置文件就可以了，学习了spring 后会堆反射有更深的理解。下面是一个读取配置的例子。

1. 新建一个text.txt

```
class=test.User
method=setUsername
```

2. 通过反射调用方法

```
public static void main(String[] args) throws Exception {
        //从text.txt中获取类名称和方法名称
        File springConfigFile = new File("D:\\Jason\\project\\hbspace\\reflect\\src\\test\\text.txt");
        Properties springConfig= new Properties();
        springConfig.load(new FileInputStream(springConfigFile));
        String className = (String) springConfig.get("class");
        String methodName = (String) springConfig.get("method");
        System.out.println(className); //输出test.User
        System.out.println(methodName); //输出 setUsername
        //根据类名称获取类对象
        Class clazz = Class.forName(className);
        //获取构造器
        Constructor c = clazz.getConstructor();
        //根据构造器，实例化出对象
        Object obj = c.newInstance();
        //根据方法名称，获取方法对象
        Method m = clazz.getMethod(methodName,String.class);
        //调用对象的指定方法
        m.invoke(obj,"哈哈哈");
        Field field = clazz.getDeclaredField("username");
        System.out.println(field.get(obj)); //输出哈哈哈
    }

```
+ 通过反射越过泛型检查
泛型是在编译期间起作用的。在编译后的.class文件中是没有泛型的。所有比如T或者E类型啊，本质都是通过Object处理的。所以可以通过使用反射来越过泛型。
```
ArrayList<String> list = new ArrayList<>();
	list.add("this");
	list.add("is");
	
	//	strList.add(5);报错
	
	/********** 越过泛型检查    **************/
	
	//获取ArrayList的Class对象，反向的调用add()方法，添加数据
	Class listClass = list.getClass(); 
	//获取add()方法
	Method m = listClass.getMethod("add", Object.class);
	//调用add()方法
	m.invoke(list, 5);
	
	//遍历集合
	for(Object obj : list){
		System.out.println(obj);
		}
	}
```