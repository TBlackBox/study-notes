
# 简介
Maven是继Ant后出现的一款基于约定优于配置（约定的一些规范无需再配置，例如：其约定好的生命周期、项目结构等。当然，Maven也提供了打破默认约定的配置办法。）原则的项目构建工具。


# 特点

1. 依赖管理：Maven能够帮助我们解决软件包依赖的管理问题，不再需要提交大量的jar包，引入第三方lib也不需要关心其依赖。
2. 规范目录结构：标准的目录结构有助于项目构建的标准化，使得项目更整洁，还可通过配置profile根据环境的不同读取不同的配置文件。
3. 可以通过每次发布都更新版本号以及统一依赖配置文件来规范软件包的发布。
4. 完整的项目构建阶段：Maven能够对项目完整阶段进行构建。
5. 支持多种插件：面向不同类型的工程项目提供不同的插件。
6. 方便集成：能够集成在IDE,eclipse中方便使用，和其他自动化构建工具也都能配合使用。
7. 可见，相比起Ant，Maven提供了更加强大和规范的功能。

# 列子
Maven基于pom（Project Object Model）进行。一个项目所有的配置都放置在pom.xml文件中，包括定义项目的类型、名字，管理依赖关系，定制插件的行为等等。
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>me.rowkey</groupId>
    <artifactId>test</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>rowkey</name>
    <url>http://maven.apache.org</url>

    <repositories>
        <repository>
            <id>nexus-suishen</id>
            <name>Nexus suishen</name>
            <url>http://maven.etouch.cn/nexus/content/groups/public/</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
                <checksumPolicy>warn</checksumPolicy>
            </snapshots>
        </repository>
    </repositories>

    <properties>
        <slf4j.version>1.7.21</slf4j.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.8.2</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
    <!--发布配置，用户名和密码需要在$M2_HOME/conf/settings.xml中配置server-->
    <distributionManagement>
        <repository>
            <id>suishen-release</id>
            <name>Suishen-Releases</name>
            <url>http://maven.etouch.cn/nexus/content/repositories/Suishen-Releases</url>
        </repository>

        <snapshotRepository>
            <id>suishen-snapshot</id>
            <name>Suishen-Snapshots</name>
            <url>http://maven.etouch.cn/nexus/content/repositories/Suishen-Snapshots</url>
        </snapshotRepository>
    </distributionManagement>
</project>
```

例子说明：  
1. Maven使用groupId:artifactId:version三者来唯一标识一个唯一的二进制版本，可以缩写为GAV。
2. packaging代表打包方式，可选的值有: pom, jar, war, ear, custom，默认为jar。
3. properties是全局属性的配置
4. dependencies是对于依赖的管理
5. plugins是对于插件的管理。

# 继承
可以通过parent实现pom的继承做统一配置管理，子pom中的配置优先级高于父pom。
```xml
?xml version=”1.0″ encoding=”UTF-8″?>

<project>

…

 <!-- 一般放到项目名字  版本号的前面 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.4.RELEASE</version>
    <relativePath/>
</parent>

…

</project>
```
下面的几个元素可以被继承：
1. groupId,version
2. Project Config
3. Dependencies
4. Plugin configuration  
值得注意的是继承可以做依赖和插件的配置管理，但是，如果子pom中没有声明和则并不生效。


# 定义项目结构
在Maven中，一个Web项目的标准结构有下面几个部分：
* src/main/java Java代码目录
* src/main/resources 配置文件目录
* src/main/webapp webapp根目录
* src/test/java 测试代码目录
* src/test/resources 测试配置目录
* target/classes 代码编译结果目标目录
* target/test-classes 测试代码编译结果目标目录

## 通过配置自定义结构
```xml
 <build>
   <plugins>
       <plugin>
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-war-plugin</artifactId>
           <configuration>
               <warSourceDirectory>WebContent/</warSourceDirectory>
           </configuration>
       </plugin>
   </plugins>
   <sourceDirectory>src</sourceDirectory>
   <testSourceDirectory>test/java</testSourceDirectory>
   <testResources>
       <testResource>
           <directory>test/resources</directory>
       </testResource>
   </testResources>
   <directory>build</directory>
</build>
```
Java代码目录移到了./src中，测试代码目录到了./test/java中，测试资源也到了./test/resources,同时编译结果目录变为了./build。此外，在maven-war-plugin中，也把Web目录的war源码目录改为了./WebContent。


# 依赖管理
一项jar包依赖可以由groupId:artifactId:version标识   
完整的标识为：groupId:artifactId:type:classifier:version  

依赖在编译部署中参与的情况可以由scope来指定, 分为: `compile`、`test`、`provided`、`system`、`import`，默认为compile。其中的import是在Maven 2.0.9后引入的，仅仅支持在中使用，导入外部的依赖版本管理。
依赖是一个树状结构，采用最近依赖原则，也可以通过`exclusions`标签来排除一些包。这里的最近依赖指的是在依赖树中，离当前结点最近的依赖优先级高，同样远时第一个优先。

## 镜像库
Maven中还有一个镜像库的配置，即在Maven的settings.xml中配置Maven镜像库。和pom.xml中的repository不同的是镜像会拦截住对远程中央库的请求，只在镜像库中进行依赖的搜索以及下载。而如果只是配置了repository，那么在repository中找不到相应的依赖时，会继续去远程中央库进行搜索和下载。
setting.xml配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
 

  <localRepository>D:\Jason\mavenku\.m2\repository</localRepository>
  <mirrors>
     <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
</settings>
```
说明：
localRepository：指定本地库位置，
mirror：指定中央库，这里用的是阿里云的。

# 构建流程
Maven的构建生命周期中几个常见phase（阶段）如下：

1. validate：验证项目以及相关信息是否正确
2. compile：编译源代码和相关资源文件
3. test：对测试代码进行测试
4. package：根据项目类型的不同进行打包
5. verify： 验证打包的正确性
6. install：将打好的包安装到本地
7. deploy：将打好的包发布到远程库中

当然，对应上述每一个phase,还有pre、post、proces做前缀的一些phase。还有一些在命令行中不常用的phase如：test-compile、integration-test等


# Profile多环境支持
现实开发中一个很常见的需求就是需要根据不同的环境打包不同的文件或者读取不同的属性值。Maven中的profile即可解决此问题。
```xml
<profiles>
   <profile>
       <id>dev</id>
       <activation>
           <activeByDefault>true</activeByDefault>
       </activation>
       <properties>
           <resources.dir>src/main/resources/dev</resources.dir>
       </properties>
   </profile>
   <profile>
       <id>test</id>
       <properties>
           <resources.dir>src/main/resources/test</resources.dir>
       </properties>
   </profile>
   <profile>
       <id>prod</id>
       <properties>
           <resources.dir>src/main/resources/prod</resources.dir>
       </properties>
   </profile>

</profiles>

<build>
    <filters>  
        <filter>${user.home}/love.properties</filter>  
    </filters>  
   <resources>
       <resource>
           <directory>${resources.dir}</directory>
           <filtering>true</filtering>  
           <includes>  
               <include>**/*</include> 
           </includes> 
        </resource>
       <resource>
           <directory>src/main/resources</directory>
           <filtering>true</filtering>  
           <includes>  
               <include>**/*</include> 
           </includes> 
       </resource>
   </resources>
</build>
```
如此，分为dev、test以及prod三种环境，对应每一种环境，其资源文件路径都不一样。在使用mvn时，使用-P参数指定profile即可生效。

此外，示例中resource下的filtering设置为true, 是为了能够在编译过程中将资源文件中的占位符替换为Maven中相应属性对应的值。例如，在resources下的config.properties文件内容：
```properties
resouceDir=${resources.dir}
```
在profile为dev时，编译结束此文件会变为:
```properties
resouceDir=src/main/resources/dev
```

而示例中的filters配置则是将外部文件的属性引入进来，同样也能够使用占位符。

如果是Web项目，想要在webapp下使用占位符，那么则需要配置maven-war-plugin:
```xml
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-war-plugin</artifactId>  
    <configuration>  
        <webResources>  
            <resource>  
                <filtering>true</filtering>  
                <directory>src/main/webapp</directory>  
                <includes>  
                    <include>**/*</include>  
                </includes>  
            </resource>  
        </webResources>  
        <warSourceDirectory>src/main/webapp</warSourceDirectory>  
        <webXml>src/main/webapp/WEB-INF/web.xml</webXml>  
    </configuration>  
</plugin>  
```

# 复用test
当需要将写的测试用例（src/test下的资源和类）以jar包形式发布出去的时候，需要用到test-jar。首先，在打包时配置maven-jar-plugin，如下：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <executions>
         <execution>
            <goals>
                <goal>test-jar</goal>
            </goals>
            <configuration>
                <excludes>
                    <exclude>*.conf</exclude>
                    <exclude>**/*.*.conf</exclude>
                    <exclude>logback.xml</exclude>
                </excludes>
            </configuration>
        </execution>
    </executions>
    <configuration>
        <excludes>
            <exclude>*.conf</exclude>
            <exclude>**/*.*.conf</exclude>
            <exclude>*.properties</exclude>
            <exclude>logback.xml</exclude>
        </excludes>
    </configuration>
</plugin>

```
使用时，指定dependency的type为test-jar：
```xml
<dependency>
    <groupId>xx</groupId>
    <artifactId>xx</artifactId>
    <version>1.0-SNAPSHOT</version>
    <type>test-jar</type>
    <scope>test</scope>
</dependency>
```

# Scala支持
Scala的官方构建工具是sbt，但是由于某些原因，在国内访问很慢。Maven有插件提供了对Scala项目的支持。
```xml
<build>
    <plugins>
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.1.6</version>
            <executions>
               <execution>
                   <id>scala-compile-first</id>
                   <phase>process-resources</phase>
                   <goals>
                       <goal>add-source</goal>
                       <goal>compile</goal>
                   </goals>
               </execution>
               <execution>
                   <id>scala-test-compile</id>
                   <phase>process-test-resources</phase>
                   <goals>
                       <goal>testCompile</goal>
                   </goals>
               </execution>
            </executions>
        </plugin>
        <plugin>
           <groupId>org.scalatest</groupId>
           <artifactId>scalatest-maven-plugin</artifactId>
           <version>1.0-RC2</version>
           <configuration>
               <reportsDirectory>${project.build.directory}/surefire-reports</reportsDirectory>
               <junitxml>.</junitxml>
               <filereports>TestSuite.txt</filereports>
               <stdout>testOutput</stdout>
           </configuration>
           <executions>
               <execution>
                   <id>test</id>
                   <goals>
                       <goal>test</goal>
                   </goals>
               </execution>
           </executions>
        </plugin>
       <plugin>
           <groupId>org.codehaus.mojo</groupId>
           <artifactId>exec-maven-plugin</artifactId>
           <version>1.2.1</version>
           <configuration>
               <executable>scala</executable>
               <arguments>
                   <argument>-classpath</argument>
                   <classpath/>
                   <argument></argument>
               </arguments>
           </configuration>
       </plugin>
</plugins>
```
net.alchim31.maven.scala-maven-plugin提供了对Scala代码的编译；org.scalatest.scalatest-maven-plugin提供了对Scala项目的测试；exec-maven-plugin配置了对Scala程序的执行。

# 常用插件
+ maven-source-plugin

源码发布插件，绑定在compile阶段，执行jar goal, 将源码以jar包的形式发布出去。

+ maven-javadoc-plugin

javadoc插件，将源码的javadoc发布出去。

+ maven-archetype-plugin

使用此插件可以定制/使用项目模板。定制模板可以遵循archetype的结构编写文件，也可以使用mvn archetype:create-from-project从一个现有的项目生成；使用模板通过archetype:generate即可。

+ maven-tomcat7-plugin

此插件可以直接使用Tomcat运行web项目，常用的命令是：mvn tomcat7:run。同样的还有jetty-maven-plugin。

+ maven-shade-plugin

此插件是maven常用打包插件，一般是将其绑定在package阶段，执行其shade goal。能够将源码和依赖的第三方资源打包在一起以供独立运行。

+ maven-assesmbly-plugin

和maven-shade-plugin一样也是打包插件，但是其功能更加强大，输出压缩包格式除了jar还支持tar、zip、gz等。

+ maven-gpg-plugin

此插件是jar包的签名插件，可以对自己发布的jar包进行签名。

# 注意
1. 在项目版本号中加入`SNAPSHOT`后缀做为快照版本可以使得Maven每次都能自动获取最新版本而无需频繁更新版本号。

2. mvn -DNAME=test可以传递给pom参数，使用${NAME}引用即可。

3. 在dependency中设置optional为true, 可使得此依赖不传递出去。如下：
```
...
<artifactId>suishen-libs</artifactId>
...

...
<dependency>
   <groupId>org.apache.httpcomponents</groupId>
   <artifactId>httpasyncclient</artifactId>
   <version>4.1.3</version>
   <optional>true</optional>
</dependency>
...
```
依赖于suishen-lib的项目除非在自己的pom里显示声明，否则不会依赖于httpasyncclient。

4. 由于Maven自定义plugin的复杂度，不够灵活，因此很多时候都是结合ant的灵活性和maven一起使用的。
```
<target name="compile" depends="clean">
  <exec executable=“mvn">
          <arg line="compile"/>
  </exec>
</target>

<target name="compile" depends="clean">
  <exec executable=“cmd">
          <arg line=“/c mvn compile"/>
  </exec>
</target>
```
日常开发中一个工程可能比较庞大，这时可以把这个工程拆分成多个子模块来管理。一个多模块工程包含一个父pom，在其中定义了它的子模块，每个子模块都是一个独立的工程。
```
<project>
    …
    <packaging>pom</packaging>

    <modules>
        <module>module-1</module>
        <module>module-2</module>
    </modules>
</project>
```
4. 可以使用第三方的takari/maven-wrapper(mvn -N io.takari:maven:wrapper -Dmaven=3.3.3)来做Maven操作（./mvnw clean），从而可以达到类似gradle wrapper的功能：不用预先安装好Maven，还能够统一项目所使用的Maven版本。

5. 打包的时候跳过测试

+ mvn命令跳过测试：
mvn install -Dmaven.test.skip=true 测试类不会生成.class 文件
mvn install -DskipTests 测试类会生成.class文件

+ 使用maven-surefire-plugin跳过测试需要在该插件配置下加：
```
<configuration> 
<skipTests>true</skipTests> 
</configuration>

```

测试类会生成.class文件

+ spring boot项目跳过测试，使用spring-boot-maven-plugin，需要在pom.xml里加：
```
<properties>
<skipTests>true</skipTests>
</properties>
```
测试类会生成.class文件