# 解决maven update project 后项目jdk变成1.5的问题


## 问题描述
在Eclipse中新建了一个Maven工程, 然后更改JDK版本为1.7, 结果每次使用Maven > Update project的时候JDK版本都恢复成1.5。


## 原因
Maven官方文档有如下描述： 编译器插件用来编译项目的源文件.从3.0版本开始, 用来编译Java源文件的默认编译器是javax.tools.JavaCompiler (如果你是用的是java 1.6) . 如果你想强制性的让插件使用javac,你必须配置插件选项 forceJavacCompilerUse. 同时需要注意的是目前source选项和target 选项的默认设置都是1.5, 与运行Maven时的JDK版本无关.如果你想要改变这些默认设置, 可以参考 Setting the -source and -target of the Java Compiler中的描述来设置 source 和target 选项. 这是Maven已知的一个特性。除非在你的POM文件中显示的指定一个版本，否则会使用编译器默认的source/target版本1.5。主要还是在于Eclipse中Maven的集成方式起到了关键作用, 它会从POM文件中生成项目的.project,.classpath以及.settings, 因此除非POM文件指定了正确的JDK版本, 否则你每次更新项目配置的时候它都会重置到1.5版本。

## 解决方案

### 方案1
在pom.xml文件中增加如下配置（可以在父的pom添加）：
```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin> 
                <groupId>org.apache.maven.plugins</groupId> 
                <artifactId>maven-compiler-plugin</artifactId> 
                <version>2.3.2</version> 
                <configuration> 
                    <source>1.8</source> 
                    <target>1.8</target> 
                    <encoding>UTF-8</encoding> 
                </configuration> 
            </plugin> 
        </plugins>
    </pluginManagement>
</build>

```

### 方案2：
在maven的setting文件中配置下面的配置：
```xml
 <profiles>
      <profile>
        <id>development</id>
        <activation>
        <jdk>1.8</jdk>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
      <maven.compiler.source>1.8</maven.compiler.source>
      <maven.compiler.target>1.8</maven.compiler.target>
      <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
  </profile>
</profiles>
```