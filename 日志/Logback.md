Logback
Logback是由Log4j创始人设计的又一个开源日志组件，相对Log4j而言，在各个方面都有了很大改进。

Logback当前分成三个模块：

+ logback-core是其它两个模块的基础模块。
+ logback-classic是Log4j的一个改良版本。logback-classic完整实现SLF4J API使你可以很方便地更换成其它日志系统如Log4j或JDK Logging。
+ logback-access访问模块与Servlet容器集成提供通过HTTP来访问日志的功能。
配置
Logback的配置文件如下：
```
<!--logback.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

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
Logback的配置文件读取顺序（默认都是读取classpath下的）：logback.groovy -> logback-test.xml -> logback.xml。如果想要自定义配置文件路径，那么只有通过修改logback.configurationFile的系统属性。
```
System.setProperty("logback.configurationFile", "...");
或者
-Dlogback.configurationFile="xx"
```
Logback的日志级别：TRACE < DEBUG < INFO < WARN < ERROR。如果logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG。

Logback中的logger同样也是有继承机制的。配置文件中的additivit也是为了不去继承rootLogger的配置，从而避免输出多份日志。

为了方便Log4j到Logback的迁移，官网提供了log4j.properties到logback.xml的转换工具：https://logback.qos.ch/translator/。

# 使用
Logback由于是天然与SLF4J集成的，因此它的使用也就是SLF4J的使用。
```
import org.slf4j.LoggerFactory;

private static final Logger LOGGER=LoggerFactory.getLogger(xx.class);

LOGGER.info(" this is a test in {}", xx.class.getName())
```
SLF4J同样支持占位符。

此外，如果想要打印json格式的日志（例如，对接日志到Logstash中），那么可以使用logstash-logback-encoder做为RollingFileAppender的encoder。
```
<encoder class="net.logstash.logback.encoder.LogstashEncoder" >
...
</encoder>
```
# 性能优化
Logback提供了AsyncAppender进行异步日志输出，此异步appender实现上利用了队列做缓冲，使得日志输出性能得到提高。
```
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
此外，由于是异步输出，为了保证日志一定会被输出以及后台线程能够被及时关闭，在应用退出时需要显示关闭logback。有两种方式：

+ 在程序退出的地方（ServletContextListener的contextDestroyed方法、Spring Bean的destroy方法）显式调用下面的代码。
```
LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
loggerContext.stop();
```
+ 在logback配置文件里，做如下配置。
```
<configuration>

    <shutdownHook class="ch.qos.logback.core.hook.DelayingShutdownHook"/>
    .... 
</configuration>
```