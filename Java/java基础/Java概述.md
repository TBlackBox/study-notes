# Java历程
1995  斯坦福大学网络公司 提出的的高级语言，最开始做小型消费电子产品。
2009.4.20 美国甲骨文公司 74亿 收购sun公司，取得版权。
含有很多库，如图形绘制，socket等等

主要特点
**一次编译  到处运行**

# JVM
是一个软件
是一个中间件
Java代码首先被编译成字节码文件，再由JVM将字节码文件翻译成机器语言，从而达到运行Java程序的目的。

# 版本
1998.12 java 1.2发布  开始使用java2
三个版本
J2SE： 标准版
J2EE：企业版
J2MC：微机版
Java 5.0后改名
Java SE    JavaEE   Java ME

# JDK 
java开发工具箱，一些列工具的集合，含有
- java编译的源码
- jvm
- 基础类库
- 打包工具
- 。。。


JDK 提供的部分工具
- Java编译器：javac.exe
- java解释器：java.exe
- Java文档生成器：javadoc.exe
- Java调试器：jdb.exe

主流JDK   sun公司的
IBM开发的  Jrocket



# Java类库
Java 官方为开发者提供了很多功能强大的类，这些类被分别放在各个包中，随JDK一起发布，称为Java类库或Java API。

API(Application Programming Interface, 应用程序编程接口)是一个通用概念。

[Java Api](https://www.oracle.com/technetwork/java/api/index.html)

Java类库中有很多包：

- 以 java.* 开头的是Java的核心包，所有程序都会使用这些包中的类；
- 以 javax.* 开头的是扩展包，x 是 extension 的意思，也就是扩展。虽然 javax.* 是对 java.* 的优化和扩展，但是由于 javax.* 使用的越来越多，很多程序都依赖于 javax.*，所以 javax.* 也是核心的一部分了，也随JDK一起发布。
- 以 org.* 开头的是各个机构或组织发布的包，因为这些组织很有影响力，它们的代码质量很高，所以也将它们开发的部分常用的类随JDK一起发布。

在包的命名方面，为了防止重名，有一个惯例：大家都以自己域名的倒写形式作为开头来为自己开发的包命名，例如百度发布的包会以 com.baidu.* 开头，w3c组织发布的包会以 org.w3c.* 开头……

组织机构的域名后缀一般为 org，公司的域名后缀一般为 com，可以认为 org.* 开头的包为非盈利组织机构发布的包，它们一般是开源的，可以免费使用在自己的产品中，不用考虑侵权问题，而以 com.* 开头的包往往由盈利性的公司发布，可能会有版权问题，使用时要注意。

java中常用的几个包介绍：

| 包名        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| java.lang   | 该包提供了Java编程的基础类，例如 Object、Math、String、StringBuffer、System、Thread等，不使用该包就很难编写Java代码了。 |
| java.util   | 该包提供了包含集合框架、遗留的集合类、事件模型、日期和时间实施、国际化和各种实用工具类（字符串标记生成器、随机数生成器和位数组）。 |
| java.io     | 该包通过文件系统、数据流和序列化提供系统的输入与输出。       |
| java.net    | 该包提供实现网络应用与开发的类。                             |
| java.sql    | 该包提供了使用Java语言访问并处理存储在数据源（通常是一个关系型数据库）中的数据API。 |
| java.awt    | 这两个包提供了GUI设计与开发的类。java.awt包提供了创建界面和绘制图形图像的所有类，而javax.swing包提供了一组“轻量级”的组件，尽量让这些组件在所有平台上的工作方式相同。 |
| javax.swing |                                                              |
| java.text   | 提供了与自然语言无关的方式来处理文本、日期、数字和消息的类和接口。 |


# import

如果你希望使用Java包中的类，就必须先使用import语句导入。

import语句与C语言中的 #include 有些类似，语法为：
  import package1[.package2…].classname;
package 为包名，classname 为类名。例如：

```java
import java.util.Date;  // 导入 java.util 包下的 Date 类
import java.util.Scanner;  // 导入 java.util 包下的 Scanner 类
import javax.swing.*;  // 导入 javax.swing 包下的所有类，* 表示所有类
```

注意：

- import 只能导入包所包含的类，而不能导入包。
- 为方便起见，我们一般不导入单独的类，而是导入包下所有的类，例如 import java.util.*;。


Java 编译器默认为所有的 Java 程序导入了 JDK 的 java.lang 包中所有的类（import java.lang.*;），其中定义了一些常用类，如 System、String、Object、Math 等，因此我们可以直接使用这些类而不必显式导入。但是使用其他类必须先导入。

前面讲到的”Hello World“程序使用了System.out.println(); 语句，System 类位于 java.lang 包，虽然我们没有显式导入这个包中的类，但是Java 编译器默认已经为我们导入了，否则程序会执行失败。

## Java类的搜索路径

Java程序运行时要导入相应的类，也就是加载 .class 文件的过程。

假设有如下的 import 语句：

```
import p1.Test;
```

该语句表明要导入 p1 包中的 Test 类。

安装JDK时，我们已经设置了环境变量 CLASSPATH 来指明类库的路径，它的值为 .;%JAVA_HOME%\lib，而 JAVA_HOME 又为 D:\Program Files\jdk1.7.0_71，所以 CLASSPATH 等价于 .;D:\Program Files\jdk1.7.0_71\lib。

Java 运行环境将依次到下面的路径寻找并载入字节码文件 Test.class：

- .p1\Test.class（"."表示当前路径）
- D:\Program Files\jdk1.7.0_71\lib\p1\Test.class


如果在第一个路径下找到了所需的类文件，则停止搜索，否则继续搜索后面的路径，如果在所有的路径下都未能找到所需的类文件，则编译或运行出错。

你可以在CLASSPATH变量中增加搜索路径，例如 .;%JAVA_HOME%\lib;C:\javalib，那么你就可以将类文件放在 C:\javalib 目录下，Java运行环境一样会找到。