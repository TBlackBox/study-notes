# Log4j2 简介
2015年8月，官方正式宣布Log4j 1.x系列生命终结，推荐大家升级到Log4j2，并号称在修正了Logback固有的架构问题的同时，改进了许多Logback所具有的功能。Log4j2与Log4j1发生了很大的变化，并不兼容。并且Log4j2不仅仅提供了日志的实现，也提供了门面，目的是统一日志框架。其主要包含两部分：

+ log4j-api： 作为日志接口层，用于统一底层日志系统
+ log4j-core : 作为上述日志接口的实现，是一个实际的日志框架

# 配置
Log4j2的配置方式只支持XML、JSON以及YAML，不再支持Properties文件,其配置文件的加载顺序如下：

+ log4j2-test.json/log4j2-test.jsn
+ log4j2-test.xml
+ log4j2.json/log4j2.jsn文件
+ log4j2.xml

如果想要自定义配置文件位置，需要设置系统属性log4j.configurationFile。
```
System.setProperty("log4j.configurationFile", "...");
或者
-Dlog4j.configurationFile="xx"
```
配置文件示例：
```
<!--log4j2.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="30">
<Appenders>
  <Console name="Console" target="SYSTEM_OUT">
    <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
  </Console>
  <File name="File" fileName="app.log" bufferedIO="true" immediateFlush="true">
    <PatternLayout>
      <pattern>%d %p %C{1.} [%t] %m%n</pattern>
    </PatternLayout>
  </File>
  <RollingFile name="RollingFile" fileName="logs/app.log"
                     filePattern="log/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
      <PatternLayout pattern="%d{yyyy-MM-dd 'at' HH:mm:ss z} %-5level %class{36} %L %M - %msg%xEx%n"/>
      <SizeBasedTriggeringPolicy size="50MB"/>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
      <DefaultRolloverStrategy max="20"/>
  </RollingFile>
</Appenders>
<Loggers>
  <logger name="myLogger" level="error" additivity="false">
    <AppenderRef ref="File" />
  </logger>
  <Root level="debug">
    <AppenderRef ref="Console"/>
  </Root>
</Loggers>
</Configuration>
```
上面的monitorInterval使得配置变动能够被实时监测并更新，且能够在配置发生改变时不会丢失任何日志事件;additivity和Log4j一样也是为了让Looger不继承父Logger的配置；Configuration中的status用于设置Log4j2自身内部的信息输出，当设置成trace时，你会看到Log4j2内部各种详细输出。

Log4j2在日志级别方面也有了一些改动：TRACE < DEBUG < INFO < WARN < ERROR < FATAL, 并且能够很简单的自定义自己的日志级别。
```
<CustomLevels>
    <CustomLevel name="NOTICE" intLevel="450" />
    <CustomLevel name="VERBOSE" intLevel="550" />
</CustomLevels>
```
上面的intLevel值是为了与默认提供的标准级别进行对照的。

# 使用
使用方式也很简单：
```
private static final Logger LOGGER = LogManager.getLogger(xx.class);

LOGGER.debug("log4j debug message");
```
这里需要注意的是其中的Logger是log4j-api中定义的接口，而Log4j1中的Logger则是类。

相比起之前我们需要先判断日志级别，再输出日志，Log4j2提供了占位符功能：
```
LOGGER.debug("error: {} ", e.getMessage());
```

# 性能优化
在性能方面，Log4j2引入了基于LMAX的Disruptor的无锁异步日志实现进一步提升异步日志的性能：
```
<AsyncLogger name="asyncTestLogger" level="trace" includeLocation="true">
    <AppenderRef ref="Console"/>
</AsyncLogger>
```
需要注意的是，由于默认日志位置信息并没有被传给异步Logger的I/O线程，因此这里的includeLocation必须要设置为true。

和Log4j一样，Log4j2也提供了缓冲配置来优化日志输出性能。
```
<Appenders>
  <File name="File" fileName="app.log" bufferedIO="true" immediateFlush="true">
    <PatternLayout>
      <pattern>%d %p %C{1.} [%t] %m%n</pattern>
    </PatternLayout>
  </File>
</Appenders>
```