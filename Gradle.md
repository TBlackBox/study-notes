


# 介绍
Gradle是一个构建系统，脚本是用groovy写的

## gradle和gradlew的区别
当把本地一个项目放入到远程版本库的时候，如果这个项目是以gradle构建的，那么其他人从远程仓库拉取代码之后如果本地没有安装过gradle会无法编译运行，如果对gradle不熟悉，会使得无法很好的去快速构建项目代码。所以gradle可以自动生成一键运行的脚本，把这些一起上传远程仓库，使得即使没有安装gradle也可以自动去安装并且编译项目代码。

Gradle是个构建系统，能够简化你的编译、打包、测试过程。熟悉Java的同学，可以把Gradle类比成Maven。
Gradle Wrapper的作用是简化Gradle本身的安装、部署。不同版本的项目可能需要不同版本的Gradle，手工部署的话比较麻烦，而且可能产生冲突，所以需要Gradle Wrapper帮你搞定这些事情。Gradle Wrapper是Gradle项目的一部分。

## 需要的环境
JDK 6 以上 ，自带Groovy库，不用安装，其他安装将会被忽略

[官网和下载地址](https://gradle.org/install/)

[w3cschool教程](https://www.w3cschool.cn/gradle_user_guide/)

## 基于两个概念

* projects ( 项目 )
代表一个jar 或者一个网页应用，或者一个ZIp压缩包
* tasks ( 任务 )
更细的分了一哈，编译一个class或者生产javdoc或者创建一个jar

# 运行一个构建

## gradle 命令会在当前目录中查找一个叫 build.gradle  
build.gradle 构建脚本

build.gradle

    task hello {
        doLast {
            println 'Hello world!'
        }
    }

上面是定义的一个任务，这个 action 是一个包含了一些 Groovy 代码的闭包

执行下面的  
    gradle -q hello
         
输出
Hello world!   
 -q  表示quiet  不生产日志信息

 ## 定义快捷任务
    task hello << {
        println 'Hello world!'
    }

跟上面效果一样 doLast 被替换成了 <<

## 案例

    task count << {
        4.times { print "$it " }
    }

输出
    > gradle -q count
    0 1 2 3
## 依赖关系
    task hello << {
        println 'Hello world!'
    }

    task intro(dependsOn: hello) << {
        println "I'm Gradle"
    }

输出
    > gradle -q intro
    Hello world!
    I'm Gradle

intro 依赖于 hello, 所以执行 intro 的时候 hello 命令会被优先执行来作为启动 intro 任务的条件.


## 动态任务
    4.times { counter ->
        task "task$counter" << {
            println "I'm task number $counter"
        }
    }

输出
    > gradle -q task1
    I'm task number 1

# java 构建入门

## java插件
如你所见, Gradle 是一种多用途的构建工具. 它可以在你的构建脚本里构建任何你想要实现的东西. 但前提是你必须先在构建脚本里加入代码, 不然它什么都不会执行.

大多数 Java 项目是非常相似的: 你需要编译你的 Java 源文件, 运行一些单元测试, 同时创建一个包含你类文件的 JAR. 如果你可以不需要为每一个项目重复执行这些步骤, 我想你会非常乐意的.

幸运的是, 你现在不再需要做这些重复劳动了. Gradle 通过使用插件解决了这个问题. 插件是 Gradle 的扩展, 它会通过某种方式配置你的项目, 典型的有加入一些预配置任务. Gradle 自带了许多插件, 你也可以很简单地编写自己的插件并和其他开发者分享它. Java 插件就是一个这样的插件. 这个插件在你的项目里加入了许多任务， 这些任务会编译和单元测试你的源文件, 并且把它们都集成一个 JAR 文件里.

Java 插件是基于合约的. 这意味着插件已经给项目的许多方面定义了默认的参数, 比如 Java 源文件的位置. 如果你在项目里遵从这些合约, 你通常不需要在你的构建脚本里加入太多东西. 如果你不想要或者是你不能遵循合约, Gradle 也允许你自己定制你的项目. 事实上, 因为对 Java 项目的支持是通过插件实现的, 如果你不想要的话, 你一点也不需要使用这个插件来构建你的项目.


## 使用Java插件

build.gradle

    apply plugin: 'java'

它将会把 Java 插件加入到你的项目中, 这意味着许多预定制的任务会被自动加入到你的项目里.

Gradle 默认在 src/main/java 目录下寻找到你的正式（生产）源码, 在 src/test/java 目录下寻找到你的测试源码, 并在src/main/resources目录下寻找到你准备打包进jar的资源文件。测试代码会被加入到环境变量中设置的目录里运行。所有的输出文件都会被创建在构建目录里, 生成的JAR文件会被存放在 build/libs 目录下.

## 建立项目
尽管Java 插件在你的项目里加入了许多任务，只有几个会在项目构建中经常用到。

最常用的任务是 build 任务, 用于完全构建你的项目.运行 gradle build 命令执行后,Gradle 将会编译和测试你的代码,并生成一个包含所有类与资源的 JAR 文件:

gradle build 命令的输出：

    > gradle build
    :compileJava
    :processResources
    :classes
    :jar
    :assemble
    :compileTestJava
    :processTestResources
    :testClasses
    :test
    :check
    :build

    BUILD SUCCESSFUL

    Total time: 1 secs

其余一些有用的任务是:

clean

删除 build 目录和所有为build生成的文件.

assemble

编译并打包你的代码, 但是并不运行单元测试.其他插件会在这个任务里加入更多的步骤.举个例子,如果你使用 War 插件,这个任务还将根据你的项目生成一个 WAR 文件.

check

编译并测试你的代码. 其他的插件会加入更多的检查步骤.举个例子, 如果你使用 checkstyle 插件, 这个任务将会运行 Checkstyle 来检查你的代码.

## 外部的依赖
通常, 一个 Java 项目的依赖许多外部的 JAR 文件.为了在项目里引用这些 JAR 文件,你需要告诉 Gradle 去哪里找它们.在 Gradle 中,JAR 文件位于一个仓库中，这里的仓库类似于 MAVEN 的仓库，可以被用来提取依赖,或者放入依赖。

举个例子，我们将使用开放的 Maven 仓库:  
build.gradle

    repositories {
        mavenCentral()
    }

接下来让我们加入一些依赖. 这里, 我们假设我们的项目在编译阶段有一些依赖:

build.gradle

    dependencies {
        compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
        testCompile group: 'junit', name: 'junit', version: '4.+'
    }
可以看到 commons-collections 被加入到了编译阶段, junit 也被加入到了测试编译阶段.

## 定制项目
Java 插件给项目加入了一些属性 (propertiy).这些属性已经被赋予了默认的值,已经足够来开始构建项目了.如果你认为不合适,改变它们的值也是很简单的.让我们看下这个例子.这里我们将指定 Java 项目的版本号,以及我们所使用的 Java 的版本.我们同样也加入了一些属性在 jar 的manifest里. 
build.gradle

    sourceCompatibility = 1.5
    version = '1.0'
    jar {
        manifest {
            attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
        }
    }

Java 插件加入的任务是常规性的任务,准确地说,就如同它们在构建文件里声明地一样. 这意味着你可以使用任务之前的章节提到的方法来定制这些任务.举个例子,你可以设置一个任务的属性,在任务里加入行为,改变任务的依赖, 或者完全重写一个任务,我们将配置一个测试任务,当测试执行的时候它会加入一个系统属性:

测试阶段加入一个系统属性

    build.gradle

    test {
        systemProperties 'property': 'value'
    }

## 发布jar
通常 JAR 文件需要在某个地方发布. 为了完成这一步, 你需要告诉 Gradle 哪里发布 JAR 文件. 在 Gradle 里, 生成的文件比如 JAR 文件将被发布到仓库里. 在我们的例子里, 我们将发布到一个本地的目录. 你也可以发布到一个或多个远程的地点.

Example 7.7. 发布 JAR 文件

build.gradle

uploadArchives {
    repositories {
       flatDir {
           dirs 'repos'
       }
    }
}
运行 gradle uploadArchives 命令来发布 JAR 文件.

# java多项目构建
现在让我们看一个典型的多项目构建. 下面是项目的布局:

Example 7.10. 多项目构建 - 分层布局

构建布局

    multiproject/
    api/
    services/webservice/
    shared/
注意: 这个例子的代码可以在 samples/java/multiproject 里找到.

现在我们能有三个项目. 项目的应用程序接口 (API) 产生一个 JAR 文件, 这个文件将提供给用户, 给用户提供基于 XML 的网络服务. 项目的网络服务是一个网络应用, 它返回 XML. shared 目录包含被 api 和 webservice 共享的代码.

为了定义一个多项目构建, 你需要创建一个设置文件 ( settings file). 设置文件放在源代码的根目录, 它指定要包含哪个项目. 它的名字必须叫做 settings.gradle. 在这个例子中, 我们使用一个简单的分层布局. 下面是对应的设置文件:

Example 7.11. 多项目构建 - settings.gradle file

settings.gradle

    include "shared", "api", "services:webservice", "services:shared"


## 通用配置
对于绝大多数多项目构建, 有一些配置对所有项目都是常见的或者说是通用的. 在我们的例子里, 我们将在根项目里定义一个这样的通用配置, 使用一种叫做配置注入的技术 (configuration injection). 这里, 根项目就像一个容器, subprojects 方法遍历这个容器的所有元素并且注入指定的配置 . 通过这种方法, 我们可以很容易的定义所有档案和通用依赖的内容清单:

Example 7.12. 多项目构建 - 通用配置

build.gradle

    subprojects {
        apply plugin: 'java'
        apply plugin: 'eclipse-wtp'

        repositories {
        mavenCentral()
        }

        dependencies {
            testCompile 'junit:junit:4.11'
        }

        version = '1.0'

        jar {
            manifest.attributes provider: 'gradle'
        }
    }
注意我们例子中, Java 插件被应用到了每一个子项目中plugin to each. 这意味着我们前几章看到的任务和属性都可以在子项目里被调用. 所以, 你可以通过在根

## 项目之间的依赖

你可以在同一个构建里加入项目之间的依赖, 举个例子, 一个项目的 JAR 文件被用来编译另外一个项目. 在 api 构建文件里我们将加入一个由 shared 项目产生的 JAR 文件的依赖. 由于这个依赖, Gradle 将确保 shared 项目总是在 api 之前被构建.

多项目构建 - 项目之间的依赖

api/build.gradle

    dependencies {
        compile project(':shared')
    }

## 创建一个发行版本
 多项目构建 - 发行文件

api/build.gradle

    task dist(type: Zip) {
        dependsOn spiJar
        from 'src/dist'
        into('libs') {
            from spiJar.archivePath
            from configurations.runtime
        }
    }

    artifacts {
    archives dist
    }

## 命令
* gradle -q projects  
列出所有子项目名称
* gradlew -q tasks   
列出所有任务  
* gradlew -q help --task libs  
指定任务的详细信息. 或者多项目构建中相同任务名称的所有任务的信息.
* gradlew dependencies   
列出项目的依赖列表, 所有依赖会根据任务区分,以树型结构展示出来.
* gradle dependencyInsight   
命令可以查看指定的依赖,例如：`gradle -q webapp:dependencyInsight * * --dependency groovy --configuration compile`
* gradle properties   
可以获取项目所有属性列表
* --profile  
 参数可以收集一些构建期间的信息并保存到 build/reports/profile 目录下. 并且会以构建时间命名这些文件.


由于时间关系，先学到这来，还有更多的东西，需要深入学习。

# 安装

centos 7
+ 创建文件夹
```
mkdir /opt/gradle
```

+ 下载
```
wget https://downloads.gradle-dn.com/distributions/gradle-5.6.1-all.zip
```

+ 解压 
```
unzip /opt/gradle gradle-5.6.1-all.zip
```

+ 配置 
```
vim /etc/profile
```
在底部添加 
```
    PATH=$PATH:/opt/gradle/gradle-5.6.1/bin
    export PATH
```
+ 让配置生效
```
source /etc/profile
```

+ 检查是否完成
```
gradle --version
```
