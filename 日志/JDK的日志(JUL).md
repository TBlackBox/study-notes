# JDK Logging
JDK Logging就是JDK自带的日志操作类，在java.util.logging包下面，通常被简称为JUL。

# 配置
JDK Logging配置文件默认位于$JAVA_HOME/jre/lib/logging.properties中，可以使用系统属性java.util.logging.config.file指定相应的配置文件对默认的配置文件进行覆盖。
```
handlers= java.util.logging.FileHandler,java.util.logging.ConsoleHandler
.handlers = java.util.logging.FileHandler,java.util.logging.ConsoleHandler #rootLogger使用的Handler
.level= INFO #rootLogger的日志级别

##以下是FileHandler的配置
java.util.logging.FileHandler.pattern = %h/java%u.log
java.util.logging.FileHandler.limit = 50000
java.util.logging.FileHandler.count = 1
java.util.logging.FileHandler.formatter =java.util.logging.XMLFormatter #配置相应的日志Formatter。

##以下是ConsoleHandler的配置
java.util.logging.ConsoleHandler.level = INFO
java.util.logging.ConsoleHandler.formatter =java.util.logging.SimpleFormatter #配置相应的日志Formatter。

#针对具体的某个logger的日志级别配置
me.rowkey.pje.log.level = SEVERE

#设置此logger不会继承成上一级logger的配置
me.rokey.pje.log.logger.useParentHandlers = false 
```
这里需要说明的是logger默认是继承的，如me.rowkey.pje.log的logger会继承me.rowkey.pje的logger配置，可以对logger配置handler和useParentHandlers（默认是为true）属性, 其中useParentHandler表示是否继承父logger的配置。

JDK Logging的日志级别比较多，从高到低为：OFF(2^31-1)—>SEVERE(1000)—>WARNING(900)—>INFO(800)—>CONFIG(700)—>FINE(500)—>FINER(400)—>FINEST(300)—>ALL(-2^31)。

# 使用
JDK Logging的使用非常简单：

```
public class JDKLogger {

	private static final Logger  logger = Logger.getLogger(JDKLogger.class.getName());
	
	public static void main(String[] args) {
		logger.info("JDK自带的INFO日志");
		logger.fine("JDK自带的FINE日志");
		logger.finer("JDK自带的FINER日志");
		logger.finest("JDK自带的FINEST日志");
		logger.log(Level.WARNING, "JDK自带的LOF日志");
	}
}

```

输出 ：
```
四月 08, 2020 5:01:12 下午 log.JDKLogger main
信息: JDK自带的INFO日志
四月 08, 2020 5:01:12 下午 log.JDKLogger main
警告: JDK自带的LOF日志
```
性能优化
JDK Logging是一个比较简单的日志框架，并没有提供异步、缓冲等优化手段。也不建议大家使用此框架。