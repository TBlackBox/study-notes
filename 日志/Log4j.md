# Log4J
Log4j应该是目前Java开发中用的最为广泛的日志框架。

# 配置
Log4j支持XML、Proerties配置，通常还是使用Properties：
```
root_log_dir=${catalina.base}/logs/app/

# 设置rootLogger的日志级别以及appender
log4j.rootLogger=INFO,default

# 设置Spring Web的日志级别
log4j.logger.org.springframework.web = ERROR

# 设置default appender为控制台输出
log4j.appender.default=org.apache.log4j.ConsoleAppender
log4j.appender.default.layout=org.apache.log4j.PatternLayout
log4j.appender.default.layout.ConversionPattern=[%-d{HH\:mm\:ss} %-3r %-5p %l] >> %m (%t)%n

# 设置新的logger，在程序中使用Logger.get("myLogger")即可使用
log4j.logger.myLogger=INFO,A2

# 设置另一个appender为按照日期轮转的文件输出
log4j.appender.A2=org.apache.log4j.DailyRollingFileAppender
log4j.appender.A2.File=${root_log_dir}log.txt
log4j.appender.A2.Append=true
log4j.appender.A2.DatePattern= yyyyMMdd'.txt'
log4j.appender.A2.layout=org.apache.log4j.PatternLayout
log4j.appender.A2.layout.ConversionPattern=[%-d{HH\:mm\:ss} %-3r %-5p %l] >> %m (%t)%n

log4j.logger.myLogger1 = INFO,A3

# 设置另一个appender为RollingFileAppender，能够限制日志文件个数
log4j.appender.A3 = org.apache.log4j.RollingFileAppender
log4j.appender.A3.Append = true
log4j.appender.A3.BufferedIO = false
log4j.appender.dA3.File = /home/popo/tomcat-yixin-pa/logs/pa.log
log4j.appender.A3.Encoding = UTF-8
log4j.appender.A3.layout = org.apache.log4j.PatternLayout
log4j.appender.A3.layout.ConversionPattern = [%-5p]%d{ISO8601}, [Class]%-c{1}, %m%n
log4j.appender.A3.MaxBackupIndex = 3 #最大文件个数
log4j.appender.A3.MaxFileSize = 1024MB
```
如果Log4j文件不直接在classpath下的话，可以使用PropertyConfigurator来进行配置：
```
PropertyConfigurator.configure("...");
```
Log4j的日志级别相对于JDK Logging来说，简化了一些：DEBUG < INFO < WARN < ERROR < FATAL。

这里的logger默认是会继承父Logger的配置（rootLogger是所有logger的父logger），如上面myLogger的输出会同时在控制台和文件中出现。如果不想这样，那么只需要如下设置:
```
log4j.additivity.myLogger=false
```
# 使用
程序中对于Log4j的使用也非常简单：
```
import org.apache.log4j.Logger;


private static final Logger LOGGER = Logger.getLogger(xx.class.getName());
...
LOGGER.info("logger info");
...
```
这里需要注意的是，虽然Log4j可以根据配置文件中日志级别的不同做不同的输出，但由于字符串创建或者拼接也是耗资源的，因此，下面的用法是不合理的。
```
LOGGER.debug("...");
```
合理的做法应该是首先判断当前的日志级别是什么，再去做相应的输出，如：
```
if(LOGGER.isDebugEnabled()){
    LOGGER.debug("...");
}
```
当然，如果是必须输出的日志可以不做此判断，比如catch异常打印错误日志的地方。

# 性能优化
Log4j为了应对某一时间里大量的日志信息进入Appender的问题提供了缓冲来进一步优化性能：
```
log4j.appender.A3.BufferedIO=true   
#Buffer单位为字节，默认是8K，IO BLOCK大小默认也是8K 
log4j.appender.A3.BufferSize=8192 
```
以上表示当日志内容达到8k时，才会将日志输出到日志输出目的地。

除了缓冲以外，Log4j还提供了AsyncAppender来做异步日志。但是AsyncAppender只能够通过xml配置使用：
```
<appender name="A2"
   class="org.apache.log4j.DailyRollingFileAppender">
   <layout class="org.apache.log4j.PatternLayout">
       <param name="ConversionPattern" value="%m%n" />
   </layout>
   <param name="DatePattern" value="'.'yyyy-MM-dd-HH" />        
   <param name="File" value="app.log" />
   <param name="BufferedIO" value="true" />
   <!-- 8K为一个写单元 -->
   <param name="BufferSize" value="8192" />
</appender>

<appender name="async" class="org.apache.log4j.AsyncAppender">
   <appender-ref ref="A2"/>
</appender>
```
