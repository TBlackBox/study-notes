## Logback
Logback是由Log4j创始人设计的又一个**开源日志组件**，相对Log4j而言，在各个方面都有了很大改进。[官方地址]( [http://logback.qos.ch](http://logback.qos.ch/))

### Logback的结构
当前分成三个模块：

+ logback-core

  它两个模块的基础模块。

+ logback-classic

  Log4j的一个改良版本。logback-classic完整实现SLF4J API使你可以很方便地更换成其它日志系统如**Log4j**或JDK **Logging**。

+ logback-access

  访问模块与Servlet容器集成提供通过HTTP来访问日志的功能(也就是说可以通过远程http的形式访问日志)。

###  取代log4j的优势
- 做到了更快的实现
- 非常充分的测试
- 很自然地实现了SLF4（这样的话，对其他日志的实现都很方便了）
- 非常详尽的官方文档
- 自动重新加载配置文件
- Lilith是log事件的观察者
- log4j的chainsaw类似、谨慎的模式和非常友好的恢复(**可以实现多个线程同时写一个日志文件**)
- 配置文件可以处理不同的情况
- Filters（过滤器）
- SiftingAppender
- 自动压缩已经打出来的log文件
- 堆栈树带有包版本
- 自动去除旧的日志文件
当然还有很多其他优势
## 配置文件

### 说明

基本配置信息都包含在configuration标签中， 需要含有至少一个**appender标签**用于指定**日志输出方式和输出格式**， root标签为**系统默认日志进程**，通过**level指定日志级别**， 通过**appender-ref关联前面指定顶的日志输出方式**。


####  **configuration**

根节点<configuration>，包含下面三个属性：

　　　　- scan: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
　　　　- scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
　　　　　　　　　　　　　　　　- debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 

　　 <!--其他配置省略--> 

</configuration>　
```
#### **子节点contextName**

用来设置上下文名称

#### **P****roperty**

子节点<property> ：用来定义变量值，它有两个属性name和value，通过<property>定义的值会被插入到logger上下文中，可以使“${}”来使用变量。

- name: 变量的名称
- value: 的值时变量定义的值

#### **timestamp**

子节点<timestamp>：获取时间戳字符串，他有两个属性key和datePattern

- key: 标识此<timestamp> 的名字；
- datePattern: 设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式。

#### **appender**

子节点<appender>：负责写日志的组件，它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名

**appender class** 类型主要有三种：
-  ConsoleAppender
-  FileAppender
-  RollingFileAppender

##### **ConsoleAppender** 
把日志输出到控制台，有以下子节点：

- <encoder>：对日志进行格式化。

详情参考https://www.cnblogs.com/ClassNotFoundException/p/6964435.html

- <target>：字符串System.out(默认)或者System.err（区别不多说了）

##### **FileAppender**
把日志添加到文件，有以下子节点：

  - <file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

  - <append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。

  - <encoder>：对记录事件进行格式化。（具体参数稍后讲解 ）

  - <prudent>：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

##### **RollingFileAppender**
滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：

- <file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

- <append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。

- <rollingPolicy>:当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类

- class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy
是最受欢迎的滚动政策，例如按天或按月。 TimeBasedRollingPolicy承担翻滚责任以及触发所述翻转的责任。TimeBasedRollingPolicy支持自动文件压缩。

 ![img](../sources\1523957-20190716170158840-2011196786.png)

有时您可能希望按日期归档文件，但同时限制每个日志文件的大小，特别是如果后处理工具对日志文件施加大小限制。为了满足这一要求，logback随附 SizeAndTimeBasedRollingPolicy。

请注意除“％d”之外的“％i”转换标记。**％i和％d令牌都是强制性的。**每当当前日志文件在当前时间段结束之前达到maxFileSize时，它将以增加的索引存档，从0开始。

#### 项目实列

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

<include resource="org/springframework/boot/logging/logback/defaults.xml"/>
<include resource="org/springframework/boot/logging/logback/console-appender.xml"/>

<property name="LOG_HOME" value="C:\log"/>
<property name="LOG_PREFIX" value="web-xxx"/>
<!--
%p:输出优先级，即DEBUG,INFO,WARN,ERROR,FATAL
%r:输出自应用启动到输出该日志讯息所耗费的毫秒数
%t:输出产生该日志事件的线程名
%f:输出日志讯息所属的类别的类别名
%c:输出日志讯息所属的类的全名
%d:输出日志时间点的日期或时间，指定格式的方式： %d{yyyy-MM-dd HH:mm:ss}
%l:输出日志事件的发生位置，即输出日志讯息的语句在他所在类别的第几行。
%m:输出代码中指定的讯息，如log(message)中的message
%n:输出一个换行符号
-->
<!--
Appender: 设置日志信息的去向,常用的有以下几个
ch.qos.logback.core.ConsoleAppender (控制台)
ch.qos.logback.core.rolling.RollingFileAppender (文件大小到达指定尺寸的时候产生一个新文件)
ch.qos.logback.core.FileAppender (文件，不推荐使用)
-->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!--&lt;!&ndash; 被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建 &ndash;&gt;-->
    <File>${LOG_HOME}/${LOG_PREFIX}-info.log</File>
    <!--<encoder>：对记录事件进行格式化。-->
    <encoder>
        <!--格式化输出：%d表示日期,后面跟时间格式，默认%data{yyyy-MM-dd}，%thread表示线程名， %msg：日志消息，%n是换行符-->
        <pattern>%date [%level] [%thread] %logger{60} [%file : %line] %msg%n</pattern>
    </encoder>
    <!--RollingFileAppender:-->
    <!--滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。-->
    <!--<rollingPolicy>:当发生滚动时，决定RollingFileAppender 的行为，涉及文件移动和重命名。-->
    <!--TimeBasedRollingPolicy： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动-->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <!--<fileNamePattern>:
    必要节点，包含文件名及“%d”转换符， “%d”可以包含一个 java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。
    如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender 的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；
    如果没设置file，活动文件的名字会根据fileNamePattern的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。-->
    <!--<fileNamePattern>${LOG_HOME}/daily/${LOG_FILE}.%d{yyyy-MM-dd}.gz</fileNamePattern>-->
    <!--压缩文件的保存路径以及保存格式，这里必须将文件压缩，.%i 必须有。如果按上面的配置会报错-->
    <fileNamePattern>${LOG_HOME}/daily/${LOG_FILE}_%d{yyyy-MM-dd}.log.%i.gz</fileNamePattern>

    <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
    <!--保存文件的大小，超过该大小自动创建新文件。旧文件压缩保存到daily目录下-->
    <maxFileSize>1MB</maxFileSize>
    </timeBasedFileNamingAndTriggeringPolicy>
    <!--&lt;!&ndash; 可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，如果是6，则只保存最近6天的文件，删除之前的旧文件 包括压缩文件 &ndash;&gt;-->
    <!--&lt;!&ndash; 每产生一个日志文件，该日志文件的保存期限天数 &ndash;&gt;-->
    <maxHistory>1</maxHistory>
    </rollingPolicy>
</appender>
    
<!--root是默认的logger 这里设定输出级别是info-->
<root level="INFO">
    <!--定义了两个appender，日志会通过往这两个appender里面写-->
    <appender-ref ref="CONSOLE"/>
    <appender-ref ref="FILE"/>
</root>
</configuration>

```



Logback的配置文件如下：
例一：

```xml
<!--logback.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<!-- 配置文件修改时重新加载，默认true -->
<configuration  scan="true">

    <property name="root_log_dir" value="${catalina.base}/logs/app/"/>

    <appender name="ROLLING_FILE_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
       <File>${root_log_dir}app.log</File>
       <Append>true</Append>
       <encoder>
           <pattern>%date [%level] [%thread] %logger{80} [%file : %line] %msg%n</pattern>
       </encoder>
       <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
           <fileNamePattern>${root_log_dir}app.log.%d{yyyy-MM-dd}.%i</fileNamePattern>
           <maxHistory>30</maxHistory> #只保留最近30天的日志文件
           <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">#每天的日志按照100MB分割
                <MaxFileSize>100MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
            <totalSizeCap>20GB</totalSizeCap>#日志总的大小上限，超过此值则异步删除旧的日志
       </rollingPolicy>
    </appender>
    
    <appender name="ROLLING_FILE_APPENDER_2" class="ch.qos.logback.core.rolling.RollingFileAppender">
       <File>${root_log_dir}mylog.log</File>
       <Append>true</Append>
       <encoder>
           <pattern>%date [%level] [%thread] %logger{80} [%file : %line] %msg%n</pattern>
       </encoder>
       #下面的日志rolling策略和ROLLING_FILE_APPENDER的等价，保留最近30天的日志，每天的日志按照100MB分隔，日志总的大小上限为20GB
       <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>mylog.log-%d{yyyy-MM-dd}.%i</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>
    </appender>
    
     <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
       <encoder>
         <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
       </encoder>
     </appender>
        
    <logger name="myLogger" level="INFO" additivity="false">
        <appender-ref ref="ROLLING_FILE_APPENDER" />
    </logger>
        
     <root level="DEBUG">          
       <appender-ref ref="STDOUT" />
     </root>  

</configuration>
```
- Appender： 这个可以理解为`追加器`，`附加者`


### root 和logger 的区别
1. root和logger是父子的关系。
Logger的appender根据参数additivity决定是否要叠加root的appender，logger的级别是其自身定义的级别，和root的级别没什么关系。
2. logger对单个包或类添加配置，相当于局部配置，root相当于全局配置
如果logger里面配置了additivity="false"，就会覆盖root的，只打印一遍；
但是additivity="true"，就会向上层再次传递，不会覆盖，而是打印两遍！

**也就是说：root 是全局配置，logger是子集配置，如果在logger里加上属性additivity="false"就会覆盖root的配置，如果为true,则全局和子集都会打印。**

**Logback的配置文件读取顺序**（默认都是读取classpath下的）：logback.groovy -> logback-test.xml -> logback.xml。如果想要自定义配置文件路径，那么只有通过修改logback.configurationFile的系统属性。

```java
//第一种方式  在代码里面设置
System.setProperty("logback.configurationFile", "...");

//第二种方式 在启动的配置类里加参数
-Dlogback.configurationFile="xx"
```
![image](../sources\1431162-20210331122913649-1230750631.png)

## Logback的日志级别

**TRACE** < **DEBUG** < **INFO** < **WARN** < **ERROR**

如果logger没有被分配级别，那么它将**从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG。**

Logback中的logger同样也是有继承机制的。配置文件中的additivit也是为了不去继承rootLogger的配置，从而避免输出多份日志。

为了方便Log4j到Logback的迁移，官网提供了log4j.properties到logback.xml的转换工具：https://logback.qos.ch/translator/。

# 使用
Logback由于是天然与SLF4J集成的，因此它的使用也就是SLF4J的使用。
```java
import org.slf4j.LoggerFactory;

private static final Logger LOGGER=LoggerFactory.getLogger(xx.class);

LOGGER.info(" this is a test in {}", xx.class.getName())
```
SLF4J同样支持占位符。

### 打印json 日志

想要打印json格式的日志（例如，对接日志到Logstash中），那么可以使用logstash-logback-encoder做为RollingFileAppender的encoder。
```java
<encoder class="net.logstash.logback.encoder.LogstashEncoder" >
...
</encoder>
```
# 性能优化
Logback提供了AsyncAppender进行异步日志输出，此异步appender实现上利用了队列做缓冲，使得日志输出性能得到提高。
```java
<appender name="FILE_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <File>${root_log_dir}app.log</File>
      <Append>true</Append>
      <encoder>
          <pattern>%date [%level] [%thread] %logger{80} [%file : %line] %msg%n</pattern>
      </encoder>
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
          <fileNamePattern>${root_log_dir}app.log.%d</fileNamePattern>
      </rollingPolicy>
</appender>
<appender name ="ASYNC" class= "ch.qos.logback.classic.AsyncAppender">  
       <discardingThreshold >0</discardingThreshold>  
       
       <queueSize>512</queueSize>  
       
       <appender-ref ref ="FILE_APPENDER"/>  
</appender>  
```
这里需要特别注意以下两个参数的配置：

+ queueSize：队列的长度,该值会影响性能，需要合理配置。
+ discardingThreshold：日志丢弃的阈值，即达到队列长度的多少会丢弃TRACT、DEBUG、INFO级别的日志，默认是80%，设置为0表示不丢弃日志。
此外，由于是异步输出，为了保证日志一定会被输出以及后台线程能够被及时关闭，在应用退出时需要**显示关闭logback**。有两种方式：

1. 在程序退出的地方（ServletContextListener的contextDestroyed方法、Spring Bean的destroy方法）显式调用下面的代码。
```java
LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
loggerContext.stop();
```
2. 在logback配置文件里，做如下配置。
```xml
<configuration>

    <shutdownHook class="ch.qos.logback.core.hook.DelayingShutdownHook"/>
    .... 
</configuration>
```



## 网上的一份配置文件说明，用于参考
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
	logback.xml的基本配置信息都包含在configuration标签中，
	需要含有至少一个appender标签用于指定日志输出方式和输出格式，
	root标签为系统默认日志进程，通过level指定日志级别，
	通过appender-ref关联前面指定顶的日志输出方式。
 -->
<!-- 定义 每隔10秒中扫描该文件 -->
<configuration scan="true" scanPeriod="10 seconds" debug="true">
    <!--定义日志输出目录-->
    <property name="LOG_HOME" value="${catalina.home}/logs/loan"/>
    <!-- 控制台输出的日志格式 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--   <pattern>[%d{yyyy-MM-dd HH:mm:ss,SSS\} %-5p] %-20c - %m%n</pattern> -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    <!-- 按照登录用户的userId产生日志 -->
    <appender name="SIFT" class="ch.qos.logback.classic.sift.SiftingAppender">
        <discriminator>
            <Key>userId</Key>
            <DefaultValue>unknown</DefaultValue>
        </discriminator>
        <sift>
            <appender name="FILE-${userId}" class="ch.qos.logback.core.rolling.RollingFileAppender">
                <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                    <fileNamePattern>${LOG_HOME}/${userId}/${HOSTNAME}_%d{yyyyMMdd}.%i.log</fileNamePattern>
                    <!-- 根据日志文件按天回滚，保存时间为30天，30天之前的都将被清理掉 -->
                    <maxHistory>7</maxHistory>
                    <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                        <maxFileSize>20MB</maxFileSize>
                    </timeBasedFileNamingAndTriggeringPolicy>
                </rollingPolicy>
                <Append>false</Append>
                <encoder>
                    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
                    <charset>UTF-8</charset>
                </encoder>
            </appender>
        </sift>
    </appender>
    <!-- 输出error log 至统一日志文件中 -->
    <appender name="ERRORAPPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/error/${HOSTNAME}_%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 根据日志文件按天回滚，保存时间为30天，30天之前的都将被清理掉 -->
            <maxHistory>7</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 只打印ERROR日志 -->
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 设置日志（访问日志，系统日志）输出位置以及格式 -->
    <appender name="INTERFACE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_HOME}/interface/${HOSTNAME}_%d{yyyy-MM-dd}.%i.log</FileNamePattern>
            <!-- 根据日志文件按天回滚，保存时间为30天，30天之前的都将被清理掉 -->
            <maxHistory>7</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>[%date] [%thread] [%level] %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 只打印INFO日志 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 输出URL访问日志 -->
    <appender name="URLLOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_HOME}/URL/${HOSTNAME}_%d{yyyy-MM-dd}.%i.log</FileNamePattern>
            <maxHistory>7</maxHistory>
            <!-- 日志大小 -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>[%thread] [%date] [%level] %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 只打印INFO日志 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- access url -->
    <appender name="mq_biz_url" class="ch.qos.logback.classic.sift.SiftingAppender">
        <discriminator>
            <Key>SYS_FLAG</Key>
            <DefaultValue>unknown</DefaultValue>
        </discriminator>
        <sift>
            <appender name="FILE-${userId}" class="ch.qos.logback.core.rolling.RollingFileAppender">
                <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                    <fileNamePattern>${LOG_HOME}/access/${SYS_FLAG}_%d{yyyyMMdd}.%i.log</fileNamePattern>
                    <maxHistory>7</maxHistory>
                    <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                        <maxFileSize>20MB</maxFileSize>
                    </timeBasedFileNamingAndTriggeringPolicy>
                </rollingPolicy>
                <Append>false</Append>
                <encoder>
                    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %msg%n</pattern>
                    <charset>UTF-8</charset>
                </encoder>
            </appender>
        </sift>
    </appender>
    <!-- 审计URL日志 -->
    <appender name="ADUIT_URL_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/aduit_logs/${HOSTNAME}_%d{yyyyMMdd}.%i.log</fileNamePattern>
            <maxHistory>7</maxHistory>
            <!-- 日志大小 -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 只打印ERROR日志 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 统一日志服务器 -->
<!--     <appender name="jinke_logs_udp" class="com.papertrailapp.logback.Syslog4jAppender"> -->
<!--         <layout class="com.jk.platform.core.common.JsonLoggerPatternLayout"> -->
<!--             <pattern>{{xdcore}} [%d{yyyy-MM-dd HH:mm:ss.SSS}][%p][%X{sessionId}][%X{traceId}][%X{cip}:%X{cPort}][%X{sip}:%X{sPort}][][%X{userId}][%t|%logger{36}|%M|%X{ctime}] - %message%n</pattern> -->
<!--         </layout> -->

<!--         <syslogConfig class="org.productivity.java.syslog4j.impl.net.udp.UDPNetSyslogConfig"> -->
<!--             <host>192.168.1.161</host> -->
<!--             <port>514</port> -->
<!--             <ident>xdcore-108</ident> -->

<!--             make logger synchronous for the tests -->
<!--             <threaded>false</threaded> -->
<!--         </syslogConfig> -->
<!--         <filter class="ch.qos.logback.classic.filter.LevelFilter"> -->
<!--             <level>WARN</level> -->
<!--             <onMatch>ACCEPT</onMatch> -->
<!--             <onMismatch>DENY</onMismatch> -->
<!--         </filter> -->
<!--     </appender> -->
    <!-- 设置异常单独打印输出 -->
    <logger name="com.jk" additivity="true">
        <level value="DEBUG"/>
        <appender-ref ref="ERRORAPPENDER"/>
    </logger>
    <logger name="com.jk.modules.platform.sysauth.session.JedisShiroSessionRepository" additivity="false">
        <level value="ERROR"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SIFT"/>
        <appender-ref ref="ERRORAPPENDER"/>
    </logger>

    <!-- 设置过滤访问日志类路径 -->
    <logger name="com.jk.modules.platform.sysauth.interceptor.ResourceInterceptor" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="INTERFACE"/>
    </logger>
    <!-- 输出URL访问日志 -->
    <logger name="com.jk.modules.common.URLInterceptor" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="URLLOG"/>
    </logger>
    <!-- 输出MQ 消费客户端日志信息 -->
    <logger name="com.jk.modules.plaform.mq.consumer.DealInfoMsgListener" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="mq_biz_url"/>
    </logger>
    <logger name="com.jk.modules.platform.common.interceptor.AuditURLInterceptor" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="ADUIT_URL_APPENDER"/>
    </logger>
    <logger name="com.jk.modules.platform.sysauth.session" additivity="false">
        <level value="DEBUG"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SIFT"/>
    </logger>
    <!-- +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ -->
    <!-- ++++++++++++++++++++++++++++ 第三方应用包日志配置 +++++++++++++++++++++++++++ -->
    <!-- +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ -->
    <!-- sql 的日志输出设置 -->
    <logger name="java.sql.Connection" additivity="false">
        <level value="DEBUG"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SIFT"/>
    </logger>
    <!-- zookeeper日志输出设置 -->
    <logger name="org.apache.zookeeper" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SIFT"/>
    </logger>
    <!-- dubbo日志输出设置 -->
    <logger name="com.alibaba.dubbo" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SIFT"/>
    </logger>
    <logger name="org.hibernate.persister.entity.AbstractEntityPersister" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SIFT"/>
    </logger>
    <!--
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder"  level="INFO" additivity="false">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="SIFT" />
    </logger>
    <logger name="org.hibernate.engine.query.HQLQueryPlan" level="INFO" additivity="false">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="SIFT" />
    </logger>
    -->
    <root level="debug">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SIFT"/>
<!--         <appender-ref ref="jinke_logs_udp"/> -->
    </root>
</configuration>
```