springboot 原本使用的都是self + logback的日志



引入依赖

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <scope>provided</scope>
</dependency>
```



配置文件配置

```yaml
logging:
  config: classpath:logback-spring.xml
```

将配置文件放到sources里

```xml
<configuration>
	<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
	<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
	<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
		
	<property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}" />
	<property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}" />

	<logger name="io.lettuce.core.protocol.CommandHandler" level="INFO"/>
	<logger name="io.lettuce.core.protocol.DefaultEndpoint" level="INFO"/>
	<logger name="io.lettuce.core.protocol.RedisStateMachine" level="INFO"/>
	<logger name="io.lettuce.core.RedisChannelHandler" level="INFO"/> 
	<logger name="io.lettuce.core.protocol.CommandEncoder" level="INFO"/>
	<logger name="org.springframework.session.web.http.SessionRepositoryFilter.SESSION_LOGGER" level="INFO"/>
	<logger name="org.springframework.data.redis.core.RedisConnectionUtils" level="INFO"/>
	<logger name="org.redisson.command.RedisExecutor" level="INFO"/>
	
	<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>${CONSOLE_LOG_PATTERN}</pattern>
		</encoder>
	</appender>

	<springProfile name="dev">
		
		<!-- 	 -->
		<logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>
	
		<root level="INFO">
			<appender-ref ref="console" />
		</root>
	</springProfile>
	
	<springProfile name="test">
		<root level="DEBUG">
			<appender-ref ref="console" />
		</root>
	</springProfile>
	
	<springProfile name="pro">
		<appender name="rolling" class="ch.qos.logback.core.rolling.RollingFileAppender">
			<file>streamer-app.log</file>
			<rollingPolicy
				class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
				<maxFileSize>100MB</maxFileSize>
				<fileNamePattern>%d{yyyy-MM-dd}-%i.zip</fileNamePattern>
				<maxHistory>60</maxHistory>
				<totalSizeCap>1GB</totalSizeCap>
			</rollingPolicy>
			<encoder>
				<pattern>${FILE_LOG_PATTERN}</pattern>
			</encoder>
		</appender>
		<!-- 生产环境 DBEUG -->
		<root level="INFO">
			<appender-ref ref="rolling" />
		</root>
	</springProfile>
</configuration>
```

