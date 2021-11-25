
https://juejin.im/post/5a6eceb8f265da3e467555fe
# 简介
Lombok是一款好用顺手的工具，就像Google Guava一样，在此予以强烈推荐，每一个Java工程师都应该使用它。Lombok是一种Java™实用工具，可用来帮助开发人员消除Java的冗长代码，尤其是对于简单的Java对象（POJO）。它通过注释实现这一目的。通过在开发环境中实现Lombok，开发人员可以节省构建诸如hashCode()和equals()这样的方法以及以往用来分类各种accessor和mutator的大量时间。

#  IntelliJ安装Lombok

setting -> plugins   搜索  lombok  安装即可



# eclipse 安装Lombok

- lombok 下载地址

```
https://projectlombok.org/download
```

1. 点击maven生成的lombok-1.16.18.jar右键，以jar的方式打开
2. 弹出如下对话框，点击【specify location……】选择eclipse.exe文件，然后点击【Install / Update】
3. .上述操作完成后，在eclipse.exe同级目录会出现lombok.jar文件且eclipse.ini文件中生成：-javaagent:lombok.jar，表明已经成功。
4. 将下面配置加入到eclipse.ini中最后一行

```
-vmargs -javaagent:lombok.jar
```
eclipse.ini的配置文件如下
```
--add-modules=ALL-SYSTEM
-javaagent:D:\Jason\eclipse\lombok.jar
-vmargs -javaagent:lombok.jar
```
5. 重启eclipse，maven重新编译一下，成功


# 导入 lombok.jar 进入工程

+ maven导入

```xml
 <dependency>
     <groupId>org.projectlombok</groupId>
     <artifactId>lombok</artifactId>
     <version>1.16.8</version>
 </dependency>
```

# Lombok注解说明

val：用在局部变量前面，相当于将变量声明为final
- @NonNull：给方法参数增加这个注解会自动在方法内对该参数进行是否为空的校验，如果为空，则抛出NPE（NullPointerException）
- @Cleanup：自动管理资源，用在局部变量之前，在当前变量范围内即将执行完毕退出之前会自动清理资源，自动生成try-finally这样的代码来关闭流
- @Getter/@Setter：用在属性上，再也不用自己手写setter和getter方法了，还可以指定访问范围
- @ToString：用在类上，可以自动覆写toString方法，当然还可以加其他参数，例如@ToString(exclude=”id”)排除id属性，或者@ToString(callSuper=true, includeFieldNames=true)调用父类的toString方法，包含所有属性
- @EqualsAndHashCode：用在类上，自动生成equals方法和hashCode方法
- @NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor：用在类上，自动生成无参构造和使用所有参数的构造函数以及把所有@NonNull属性作为参数的构造函数，如果指定staticName = “of”参数，同时还会生成一个返回类对象的静态工厂方法，比使用构造函数方便很多
- @Data：注解在类上，相当于同时使用了@ToString、@EqualsAndHashCode、@Getter、- - --
@Setter和@RequiredArgsConstrutor这些注解，对于POJO类十分有用
- @Value：用在类上，是@Data的不可变形式，相当于为属性添加final声明，只提供getter方法，而不提供setter方法
- @Builder：用在类、构造器、方法上，为你提供复杂的builder APIs，让你可以像如下方式一样调用Person.builder().name("Adam Savage").city("San Francisco").job("Mythbusters").job("Unchained Reaction").build();更多说明参考Builder
- @SneakyThrows：自动抛受检异常，而无需显式在方法上使用throws语句
- @Synchronized：用在方法上，将方法声明为同步的，并自动加锁，而锁对象是一个私有的属性$lock或$LOCK，而java中的synchronized关键字锁对象是this，锁在this或者自己的类对象上存在副作用，就是你不能阻止非受控代码去锁this或者类对象，这可能会导致竞争条件或者其它线程错误
- @Getter(lazy=true)：可以替代经典的Double Check Lock样板代码
- @Log：根据不同的注解生成不同类型的log对象，但是实例名称都是log，有六种可选实现类
```java
@CommonsLog Creates log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);
@Log Creates log = java.util.logging.Logger.getLogger(LogExample.class.getName());
@Log4j Creates log = org.apache.log4j.Logger.getLogger(LogExample.class);
@Log4j2 Creates log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);
@Slf4j Creates log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
@XSlf4j Creates log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);
```
## `@RequiredArgsConstructor`注解
### 使用
定义到类上
### 作用
生成有必须参数的构造函数，必需的参数是最终字段和具有约束的字段，例如@NonNull 。
### 要点
1. 必须声明的变量魏final.
2. 根据构造器注入的，相当于当容器调用带有一组参数的类构造函数时，基于构造函数的 DI 就完成了，其中每个参数代表一个对其他类的依赖。


# 3.2 Lombok代码示例

val示例
```java
public static void main(String[] args) {
    val sets = new HashSet<String>();
    val lists = new ArrayList<String>();
    val maps = new HashMap<String, String>();
    //=>相当于如下
    final Set<String> sets2 = new HashSet<>();
    final List<String> lists2 = new ArrayList<>();
    final Map<String, String> maps2 = new HashMap<>();
}
```
复制代码
@NonNull示例
```JAVA
public void notNullExample(@NonNull String string) {
    string.length();
}
//=>相当于
public void notNullExample(String string) {
    if (string != null) {
        string.length();
    } else {
        throw new NullPointerException("null");
    }
}
```

@Cleanup示例

```java
public static void main(String[] args) {
    try {
        @Cleanup InputStream inputStream = new FileInputStream(args[0]);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
    //=>相当于
    InputStream inputStream = null;
    try {
        inputStream = new FileInputStream(args[0]);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } finally {
        if (inputStream != null) {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

@Getter/@Setter示例

```java
@Setter(AccessLevel.PUBLIC)
@Getter(AccessLevel.PROTECTED)
private int id;
private String shap;
```

@ToString示例
```java
@ToString(exclude = "id", callSuper = true, includeFieldNames = true)
public class LombokDemo {
    private int id;
    private String name;
    private int age;
    public static void main(String[] args) {
        //输出LombokDemo(super=LombokDemo@48524010, name=null, age=0)
        System.out.println(new LombokDemo());
    }
}
```

@EqualsAndHashCode示例

```java
@EqualsAndHashCode(exclude = {"id", "shape"}, callSuper = false)
public class LombokDemo {
    private int id;
    private String shap;
}
```

@NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor示例
```java
@NoArgsConstructor
@RequiredArgsConstructor(staticName = "of")
@AllArgsConstructor
public class LombokDemo {
    @NonNull
    private int id;
    @NonNull
    private String shap;
    private int age;
    public static void main(String[] args) {
        new LombokDemo(1, "circle");
        //使用静态工厂方法
        LombokDemo.of(2, "circle");
        //无参构造
        new LombokDemo();
        //包含所有参数
        new LombokDemo(1, "circle", 2);
    }
}

```

@Data示例
```java
import lombok.Data;
@Data
public class Menu {
    private String shopId;
    private String skuMenuId;
    private String skuName;
    private String normalizeSkuName;
    private String dishMenuId;
    private String dishName;
    private String dishNum;
    //默认阈值
    private float thresHold = 0;
    //新阈值
    private float newThresHold = 0;
    //总得分
    private float totalScore = 0;
}
```
@Value示例
```java
@Value
public class LombokDemo {
    @NonNull
    private int id;
    @NonNull
    private String shap;
    private int age;
    //相当于
    private final int id;
    public int getId() {
        return this.id;
    }
    ...
}
```

@Builder示例
```java
@Builder
public class BuilderExample {
    private String name;
    private int age;
    @Singular
    private Set<String> occupations;
    public static void main(String[] args) {
        BuilderExample test = BuilderExample.builder().age(11).name("test").build();
    }
}
```

@SneakyThrows示例
```java
import lombok.SneakyThrows;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
public class Test {
    @SneakyThrows()
    public void read() {
        InputStream inputStream = new FileInputStream("");
    }
    @SneakyThrows
    public void write() {
        throw new UnsupportedEncodingException();
    }
    //相当于
    public void read() throws FileNotFoundException {
        InputStream inputStream = new FileInputStream("");
    }
    public void write() throws UnsupportedEncodingException {
        throw new UnsupportedEncodingException();
    }
}
```

@Synchronized示例
```java
public class SynchronizedDemo {
    @Synchronized
    public static void hello() {
        System.out.println("world");
    }
    //相当于
    private static final Object $LOCK = new Object[0];
    public static void hello() {
        synchronized ($LOCK) {
            System.out.println("world");
        }
    }
}
```
@Getter(lazy = true)
```java
public class GetterLazyExample {
    @Getter(lazy = true)
    private final double[] cached = expensive();
    private double[] expensive() {
        double[] result = new double[1000000];
        for (int i = 0; i < result.length; i++) {
            result[i] = Math.asin(i);
        }
        return result;
    }
}
```

// 相当于如下所示: 
```
import java.util.concurrent.atomic.AtomicReference;
public class GetterLazyExample {
    private final AtomicReference<java.lang.Object> cached = new AtomicReference<>();
    public double[] getCached() {
        java.lang.Object value = this.cached.get();
        if (value == null) {
            synchronized (this.cached) {
                value = this.cached.get();
                if (value == null) {
                    final double[] actualValue = expensive();
                    value = actualValue == null ? this.cached : actualValue;
                    this.cached.set(value);
                }
            }
        }
        return (double[]) (value == this.cached ? null : value);
    }
    private double[] expensive() {
        double[] result = new double[1000000];
        for (int i = 0; i < result.length; i++) {
            result[i] = Math.asin(i);
        }
        return result;
    }
}
```
复制代码4 Lombok注解原理
说道 Lombok，我们就得去提到 JSR 269: Pluggable Annotation Processing API (www.jcp.org/en/jsr/deta…) 。JSR 269 之前我们也有注解这样的神器，可是我们比如想要做什么必须使用反射，反射的方法局限性较大。首先，它必须定义@Retention为RetentionPolicy.RUNTIME，只能在运行时通过反射来获取注解值，使得运行时代码效率降低。其次，如果想在编译阶段利用注解来进行一些检查，对用户的某些不合理代码给出错误报告，反射的使用方法就无能为力了。而 JSR 269 之后我们可以在 Javac的编译期利用注解做这些事情。所以我们发现核心的区分是在 运行期 还是 编译期。

从上图可知，Annotation Processing 是在解析和生成之间的一个步骤。具体详细步骤如下：

上图是 Lombok 处理流程，在Javac 解析成抽象语法树之后(AST), Lombok 根据自己的注解处理器，动态的修改 AST，增加新的节点(所谓代码)，最终通过分析和生成字节码。
自从Java 6起，javac就支持“JSR 269 Pluggable Annotation Processing API”规范，只要程序实现了该API，就能在javac运行的时候得到调用。


常用的项目管理工具Maven所使用的java编译工具来源于配置的第三方工具，如果我们配置这个第三方工具为Oracle javac的话，那么Maven也就直接支持lombok了;
Intellij Idea配置的编译工具为Oracle javac的话，也就直接支持lombok了;


IDE工具问题解决：

现在有一个A类，其中有一些字段，没有创建它们的setter和getter方法，使用了lombok的@Data注解，另外有一个B类，它调用了A类实例的相应字段的setter和getter方法
编译A类和B类所在的项目，并不会报错，因为最终生成的A类字节码文件中存在相应字段的setter和getter方法
但是，IDE发现B类源代码中所使用的A类实例的setter和getter方法在A类源代码中找不到定义，IDE会认为这是错误
要解决以上这个不是真正错误的错误，可以下载安装Intellij Idea中的"Lombok plugin"。

5 自定义支持JSR269的注解
一般javac的编译过程，java文件首先通过进行解析构建出一个AST，然后执行注解处理，最后经过分析优化生成二进制的.class文件。我们能做到的是，在注解处理阶段进行一些相应处理。首先我们在META-INF.services下创建如下文件：

文件中指定我们的注解处理器：com.alipay.kris.other.lombok.MyAnnotaionProcessor，然后我们接可以编写自己的注解处理器，一个简单的实例代码如下：
```JAVA
@SupportedSourceVersion(SourceVersion.RELEASE_8)
@SupportedAnnotationTypes("com.alipay.kris.other.lombok.*")
public class MyAnnotaionProcessor extends AbstractProcessor {
    public MyAnnotaionProcessor() {
        super();
    }
    @Override
    public boolean process(Set<? extends TypeElement> annotations,RoundEnvironment roundEnv) {
        for (Element elem : roundEnv.getElementsAnnotatedWith(MyAnnotation.class)) {
            MyAnnotation annotation = elem.getAnnotation(MyAnnotation.class);
            String message = "annotation found in " + elem.getSimpleName()
                + " with  " + annotation.value();
            addToString(elem);
            processingEnv.getMessager().printMessage(Diagnostic.Kind.NOTE, message);
        }
        return true; // no further processing of this annotation type
    }
}
```