# 简介

## Gson 特点
- Gson是目前功能最全的Json解析神器，Gson当初是为因应Google公司内部需求而由Google自行研发而来，但自从在2008年五月公开发布第一版后已被许多公司或用户应用。
- Gson的应用主要为toJson与fromJson两个转换函数，无依赖，不需要例外额外的jar，能够直接跑在JDK上。
- 而在使用这种对象转换之前需先创建好对象的类型以及其成员才能成功的将JSON字符串成功转换成相对应的对象。
- 类里面只要有get和set方法，Gson完全可以将复杂类型的json到bean或bean到json的转换，是JSON解析的神器。
- Gson在功能上面无可挑剔，但是性能上面比FastJson有所差距。

## Fastjson的特点
- Fastjson是一个Java语言编写的高性能的JSON处理器,由阿里巴巴公司开发。
- 无依赖，不需要例外额外的jar，能够直接跑在JDK上。
- FastJson在复杂类型的Bean转换Json上会出现一些问题，可能会出现引用的类型，导致Json转换出错，需要制定引用。
- FastJson采用独创的算法，将parse的速度提升到极致，超过所有json库。


在项目选型的时候可以使用Google的Gson和阿里巴巴的FastJson两种并行使用，
如果只是功能要求，没有性能要求，可以使用google的Gson，
如果有性能上面的要求可以使用Gson将bean转换json确保数据的正确，使用FastJson将Json转换Bean


# Gson的资料地址
[api文档](https://www.javadoc.io/doc/com.google.code.gson/gson/latest/com.google.gson/module-summary.html)
[github地址](https://github.com/google/gson)
[用法指导](https://github.com/google/gson/blob/master/UserGuide.md)

# Gson的几个核心库
Gson类：解析json的最基础的工具类
GsonBuilder类：解析json的最基础的工具类

JsonParser类：解析器来解析JSON字符串到JsonElements的解析树

JsonElement类：一个类代表的JSON元素抽象类，下面4个类都继承了他
- JsonObject类：JSON对象类型
- JsonArray类：JsonObject数组
- JsonNull类： null值
- JsonPrimitive类：可以理解为基础类型

TypeToken类：用于创建type，比如泛型List<?>

# maven引入
```
<dependencies>
    <!--  Gson: Java to Json conversion -->
    <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.8.6</version>
      <scope>compile</scope>
    </dependency>
</dependencies>
```

# 基本类型序列换和反序列化
```
//基本类型序列化
Gson gson = new Gson();
gson.toJson(1); // 1       
gson.toJson("abcd"); //"abcd"      
gson.toJson(new Long(10));// 10
gson.toJson(new int[]{1,2,3,4});  //[1,2,3,4]
gson.toJson(new String[] {"aa","bb","cc"}); //["aa","bb","cc"]

// 基本类型反序列化
int one = gson.fromJson("1", int.class);  //1
Integer one1 = gson.fromJson("1", Integer.class); //1
Long one2 = gson.fromJson("1", Long.class); //1
Boolean b = gson.fromJson("false", Boolean.class); //false
String str = gson.fromJson("\"abc\"", String.class); // abc
String[] anotherStr = gson.fromJson("[\"abc\"]", String[].class); //["abc"]
```

# 对象的序列化和反序列化
```
public class User{
    private String name;
    private Integer age;
    private Boolean isBoy;
    public User(String name,Integer age,Boolean isBoy) {
        this.age = age;
        this.name = name;
        this.isBoy = isBoy;
    }
}
```
## 序列化和反序列化
```
User user = new User("张三", 25, true);
//序列化对象
Gson gson = new Gson();
String userStr = gson.toJson(user);
//{"name":"张三","age":25,"isBoy":true}
System.out.println(userStr);

//反序列化
User user2 = gson.fromJson(userStr, User.class);
```
** 说明 **
- 对象推荐使用私有的。
- 不需要任何注解来修饰需要序列化和反序列化的字段，默认全部进行序列化和反序列化，当然包含超类。
- 如果字段被标记被暂时的，则会被忽略。
- 序列化时，一个null字段将会被忽略。
- 反序列化时，json缺少的条目将被赋予默认值：
	- 对象  赋值为null
	- 数字  赋值为0
	- 布尔  赋值为false
- 如果字段是组合类型，将不会被序列化和反序列化。
- 与内部类，匿名类和局部类中的外部类相对应的字段将被忽略，并且不包含在序列化或反序列化中。 

- Gson还可以反序列化静态嵌套类。但是，Gson无法自动反序列化纯内部类，因为它们的无参构造函数还需要引用包含对象，而该对象在反序列化时不可用。您可以通过使内部类静态化或为其提供自定义InstanceCreator来解决此问题。这是一个例子：
```
public class A { 
  public String a; 

  class B { 

    public String b; 

    public B() {
      // No args constructor for B
    }
  } 
}
```
** 注意 **
上述B类（默认情况下）无法使用Gson进行序列化。
由于类B是内部类，因此Gson不能将{“ b”：“ abc”}反序列化为B的实例。如果将其定义为静态B类，则Gson将能够反序列化该字符串。另一种解决方案是为B编写自定义实例创建者。
```
public class InstanceCreatorForB implements InstanceCreator<A.B> {
  private final A a;
  public InstanceCreatorForB(A a)  {
    this.a = a;
  }
  public A.B createInstance(Type type) {
    return a.new B();
  }
}
```
这只是一种做法，但不推荐。

# 集合的序列化和反序列化
```
Gson gson = new Gson();
Collection<Integer> ints = new ArrayList();
ints.add(1);
ints.add(2);
ints.add(3);
ints.add(4);
ints.add(5);

//序列换
String json = gson.toJson(ints);  // [1,2,3,4,5]

// 反序列化
Type collectionType = new TypeToken<Collection<Integer>>(){}.getType();
Collection<Integer> ints2 = gson.fromJson(json, collectionType);
```
** 注意 **
通过`TypeToken`传递泛型
```
Type type = new TypeToken<T>(){}.getType();
```
定义的泛型Y,就是JSON序列化和反序列化的Java类型，当然，里面可以是复杂的集合类型。

- Gson可以序列化任意集合，但是不能进行反序列化他们，因为用户无法指示生成的对象类型。
- 反序列化时，集合必须是特定的通用类型。

# 泛型
当您调用toJson（obj）时，Gson会调用obj.getClass（）以获取有关要序列化的字段的信息。同样，通常可以在fromJson（json，MyClass.class）方法中传递MyClass.class对象。如果对象是非泛型类型，则可以正常工作。但是，如果对象属于泛型，则由于Java Type Erasure，泛型信息会丢失。这是说明要点的示例：
```
class Foo<T> {
  T value;
}
Gson gson = new Gson();
Foo<Bar> foo = new Foo<Bar>();
gson.toJson(foo); // 可能会失败  

gson.fromJson(json, foo.getClass()); // 失败反序列化 foo.value 作为 Bar
```
** 分析 ** 
上面的代码无法将值解释为Bar类型，因为Gson调用foo.getClass（）来获取其类信息，但是此方法返回的是原始类Foo.class。这意味着Gson无法知道这是Foo <Bar>类型的对象，而不仅仅是纯Foo。
可以通过为泛型类型指定正确的参数化类型来解决此问题。您可以通过使用TypeToken类来实现。

```
Type fooType = new TypeToken<Foo<Bar>>() {}.getType();
gson.toJson(foo, fooType);

gson.fromJson(json, fooType);
```
用于获取fooType的习惯用法实际上定义了一个匿名的本地内部类，其中包含一个方法getType（），该方法返回完全参数化的类型。

# 序列化和反序列化带有任意对象的集合

如果要序列化和反序列化`["hello",5,{"name":"GREETINGS","source":"guest"}]`,
您可以使用Gson序列化集合，而无需执行任何特定操作：toJson（collection）将写出所需的输出。但是，由于Gson无法知道如何将输入映射到类型，因此fromJson（json，Collection.class）的反序列化将不起作用。Gson要求您在fromJson（）中提供集合类型的通用版本。
可以有下面的3种方法

1. 为Collection.class注册一个类型适配器，该适配器查看每个数组成员并将它们映射到适当的对象。这种方法的缺点是它将破坏Gson中其他收集类型的反序列化。
2. `为MyCollectionMemberType`注册类型适配器，并将`fromJson（）`与`Collection <MyCollectionMemberType>`一起使用。
```
仅当数组显示为顶级元素或可以将保存集合的字段类型更改为Collection 	<MyCollectionMemberType>类型时，此方法才实用。
```
3. 使用Gson的解析器API（低级流解析器或DOM解析器JsonParser）解析数组元素，然后对每个数组元素使用Gson.fromJson（），这是首选方法。下面是案列。
```
public class RawCollectionsExample {
  static class Event {
    private String name;
    private String source;
    private Event(String name, String source) {
      this.name = name;
      this.source = source;
    }
    @Override
    public String toString() {
      return String.format("(name=%s, source=%s)", name, source);
    }
  }

  @SuppressWarnings({ "unchecked", "rawtypes" })
  public static void main(String[] args) {
    Gson gson = new Gson();
    Collection collection = new ArrayList();
    collection.add("hello");
    collection.add(5);
    collection.add(new Event("GREETINGS", "guest"));
    String json = gson.toJson(collection);
    
    System.out.println("序列化后的字符串: " + json);
    
    JsonParser parser = new JsonParser();
    JsonArray array = parser.parse(json).getAsJsonArray();
    
    String message = gson.fromJson(array.get(0), String.class);
    int number = gson.fromJson(array.get(1), int.class);
    Event event = gson.fromJson(array.get(2), Event.class);
    System.out.printf("反序列化后得到的信息: %s, %d, %s", message, number, event);
  }
}
```
输出
```
序列化集合: ["hello",5,{"name":"GREETINGS","source":"guest"}]
Using Gson.fromJson() to get: hello, 5, (name=GREETINGS, source=guest)
```
# 内置序列化器和反序列化器
- java.net.URL以将其与“ https://github.com/google/gson/”之类的字符串进行匹配
- java.net.URI以将其与“ / google / gson /”之类的字符串进行匹配有关
更多信息，请参见内部类[TypeAdapters](https://github.com/google/gson/blob/master/gson/src/main/java/com/google/gson/internal/bind/TypeAdapters.java)。您也可以在此页面上找到一些常用类的源代码，例如[JodaTime](https://sites.google.com/site/gson/gson-type-adapters-for-common-classes-1)。

# 自定义序列化和反序列化
有的时候默认的不是你想要的，例如`DateTime`等，Gson允许自定义。分为两部分：
1. Json序列化器：需要为对象定义自定义序列化器。
2. Json 反序列化器：需要为类型自定义反序列化器。
3. 实例创建者：如果有无参构造器或注册了反序列化器，则不需要。
```
GsonBuilder gson = new GsonBuilder();
gson.registerTypeAdapter(MyType2.class, new MyTypeAdapter());
gson.registerTypeAdapter(MyType.class, new MySerializer());
gson.registerTypeAdapter(MyType.class, new MyDeserializer());
gson.registerTypeAdapter(MyType.class, new MyInstanceCreator());
```
registerTypeAdapter调用检查类型适配器是否实现了多个这些接口之一，并为所有这些接口注册它。

## 创建一个`DateTime`序列化器
```
private class DateTimeSerializer implements JsonSerializer<DateTime> {
  public JsonElement serialize(DateTime src, Type typeOfSrc, JsonSerializationContext context) {
    return new JsonPrimitive(src.toString());
  }
}
```

## 创建一个`DateTime`反序列化器
```
private class DateTimeDeserializer implements JsonDeserializer<DateTime> {
  public DateTime deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context)
      throws JsonParseException {
    return new DateTime(json.getAsJsonPrimitive().getAsString());
  }
}
```

## 实例创建
反序列化对象时，Gson需要创建该类的默认实例。用于序列化和反序列化的行为良好的类应具有无参数的构造函数,如果没得的话，就需要创建下面的实例。
```
private class MoneyInstanceCreator implements InstanceCreator<Money> {
  public Money createInstance(Type type) {
    return new Money("1000000", CurrencyCode.USD);
  }
}
```

# Json输出格式
Gson默认的是紧凑型的格式（意思就是没得缩进这些，为null的字段不会输出），如果您想使用有格式的输出功能，则必须使用GsonBuilder配置Gson实例。JsonFormatter不会通过我们的公共API公开，因此客户端无法为JSON输出配置默认的打印设置/边距。目前，我们仅提供默认的JsonPrintFormatter，其默认行长为80个字符，2个字符的缩进和4个字符的右边距。

以下示例显示了如何配置Gson实例以使用默认的`JsonPrintFormatter`（有格式的输出）而不是`JsonCompactFormatter`(紧凑的输出)：
```
Gson gson = new GsonBuilder().setPrettyPrinting().create();
String jsonOutput = gson.toJson(someObject);
```

# 空对象的支持
Gson中实现的默认行为是忽略空对象字段。这允许更紧凑的输出格式。但是，当JSON格式转换回其Java形式时，客户端必须为这些字段定义默认值。上面对象的时候提过。
这是配置Gson实例以输出null的方法：
```
Gson gson = new GsonBuilder().serializeNulls().create();
```
注意：使用Gson序列化null时，它将在JsonElement结构中添加一个JsonNull元素。因此，可以在自定义序列化/反序列化中使用此对象。
例如
```
public class Foo {
  private final String s;
  private final int i;

  public Foo() {
    this(null, 5);
  }

  public Foo(String s, int i) {
    this.s = s;
    this.i = i;
  }
}

Gson gson = new GsonBuilder().serializeNulls().create();
Foo foo = new Foo();
String json = gson.toJson(foo);
System.out.println(json);

json = gson.toJson(null);
System.out.println(json);
```
输出
```
{"s":null,"i":5}
null
```

# 版本支持
可以使用`@Since`批注维护同一对象的多个版本。可以在类，字段以及将来的方法中使用此注释。为了利用此功能，您必须将Gson实例配置为忽略大于某个版本号的任何字段/对象。如果在Gson实例上未设置任何版本，则它将对所有字段和类进行序列化和反序列化，而与版本无关。
例如：
```
public class VersionedClass {
  @Since(1.1) private final String newerField;
  @Since(1.0) private final String newField;
  private final String field;

  public VersionedClass() {
    this.newerField = "newer";
    this.newField = "new";
    this.field = "old";
  }
}

VersionedClass versionedObject = new VersionedClass();
Gson gson = new GsonBuilder().setVersion(1.0).create();
String jsonOutput = gson.toJson(versionedObject);
System.out.println(jsonOutput);
System.out.println();

gson = new Gson();
jsonOutput = gson.toJson(versionedObject);
System.out.println(jsonOutput);
```
输出
```
{"newField":"new","field":"old"}

{"newerField":"newer","newField":"new","field":"old"}
```

# 排除序列化和反序列化的字段
Gson可以排除顶级类，字段，字段类型，下面是几种方式
1. Java编辑排除
默认情况下，如果将字段标记为暂时的，则将其排除。同样，如果将字段标记为静态，则默认情况下将其排除。如果要包括一些临时字段，则可以执行以下操作：
```
import java.lang.reflect.Modifier;
Gson gson = new GsonBuilder()
    .excludeFieldsWithModifiers(Modifier.STATIC)
    .create();
```
注意：您可以将任意数量的Modifier常量提供给excludeFieldsWithModifiers方法。例如：
```
Gson gson = new GsonBuilder()
    .excludeFieldsWithModifiers(Modifier.STATIC, Modifier.TRANSIENT, Modifier.VOLATILE)
    .create();
```

2. 使用注解`@Expose`排除
可以在字段上使用注解`@Expose`,排除对应的字段，但注意的是，必须使用下面的创建gson实例才有效。
```
new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()
```

3. 自定义排除策略
可以使用自定义的排除策略，更多看[ExclusionStrategy](https://www.javadoc.io/doc/com.google.code.gson/gson/2.8.5/com/google/gson/ExclusionStrategy.html)
下面的示例演示如何排除标记有特定@Foo注释的字段，以及如何排除String类的顶级类型（或声明的字段类型）。
```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD})
public @interface Foo {
  // Field tag only annotation
}

public class SampleObjectForTest {
  @Foo private final int annotatedField;
  private final String stringField;
  private final long longField;
  private final Class<?> clazzField;

  public SampleObjectForTest() {
    annotatedField = 5;
    stringField = "someDefaultValue";
    longField = 1234;
  }
}

public class MyExclusionStrategy implements ExclusionStrategy {
  private final Class<?> typeToSkip;

  private MyExclusionStrategy(Class<?> typeToSkip) {
    this.typeToSkip = typeToSkip;
  }

  public boolean shouldSkipClass(Class<?> clazz) {
    return (clazz == typeToSkip);
  }

  public boolean shouldSkipField(FieldAttributes f) {
    return f.getAnnotation(Foo.class) != null;
  }
}

public static void main(String[] args) {
  Gson gson = new GsonBuilder()
      .setExclusionStrategies(new MyExclusionStrategy(String.class))
      .serializeNulls()
      .create();
  SampleObjectForTest src = new SampleObjectForTest();
  String json = gson.toJson(src);
  System.out.println(json);
}
```

输出
```
{"longField":1234}
```

# 字段命名支持
可以通过` @SerializedName`注解重新命名字段
```
private class SomeObject {
  @SerializedName("custom_naming") private final String someField;
  private final String someOtherField;

  public SomeObject(String a, String b) {
    this.someField = a;
    this.someOtherField = b;
  }
}

SomeObject someObject = new SomeObject("first", "second");
Gson gson = new GsonBuilder().setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE).create();
String jsonRepresentation = gson.toJson(someObject);
System.out.println(jsonRepresentation);
```
输出
```
{"custom_naming":"first","SomeOtherField":"second"}
```
参考[FieldNamingPolicy](https://www.javadoc.io/doc/com.google.code.gson/gson/2.8.5/com/google/gson/FieldNamingPolicy.html)里面的类型值。

# 总结 

1. Gson的几个注解
 - `@Expose`
 字段的排除，序列化和反序列化对该字段不生效，必须配置下面的才生效。
 ```
 Gson gson = new GsonBuilder()
 .excludeFieldsWithModifiers(Modifier.STATIC, Modifier.TRANSIENT, Modifier.VOLATILE)
 .create();
 ```
 - `@SerializedName`
重命名字段
- `@Since`
对版本的支持，用于指定版本,配合下面的使用
```
Gson gson = new GsonBuilder().setVersion(1.0).create();
```
- `@JsonAdapter`
指示用于类或字段的Gson TypeAdapter的注释。

更多东西看官方文档和分析源码。