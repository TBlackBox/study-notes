## spring两种定时任务解决方案
- Spring Framework的Spring Task模块，提供了轻量级的定时任务的实现
- Spring Boot 2.0 版本，整合Quartz作业调度框架，提供了功能强大的定时任务实现。（spring boot 1.x未提供Quartz的自动化配置，2.x 提供了支持）

## 优秀的开源的任务中间件
- Elastic-Job ： 唯品会基于Elastic-Job之上，演化了Saturn项目
- Apache DolphinScheduler
- XXL-Job
目前Elastic-Job和xxl-Job为主，使用xxl-job的团队多一点，上手比较容易，运维功能更为完善。

# Spring Task
[官方文档地址](https://docs.spring.io/spring-framework/docs/5.2.x/spring-framework-reference/integration.html#scheduling)

Spring Task 是 Spring Framework 的模块，所以在我们引入 spring-boot-starter-web 依赖后，无需特别引入它。

同时，考虑到我们希望让项目启动时，不自动结束 JVM 进程，所以我们引入了 spring-boot-starter-web 依赖。
在文件文件类上配置即可
```java
@Configuration
@EnableScheduling
public class ScheduleConfiguration {
}
```

创建任务类
```java
@Component
public class DemoJob {

    private Logger logger = LoggerFactory.getLogger(getClass());

    private final AtomicInteger counts = new AtomicInteger();

    @Scheduled(fixedRate = 2000)
    public void execute() {
        logger.info("[execute][定时第 ({}) 次执行]", counts.incrementAndGet());
    }

}
```
启动spring 项目即可运行

## @Scheduled

[`@Scheduled`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/scheduling/annotation/Scheduled.java) 注解，设置定时任务的执行计划。

**常用**属性如下：

- `cron` 属性：Spring Cron 表达式。例如说，`"0 0 12 * * ?"` 表示每天中午执行一次，`"11 11 11 11 11 ?"` 表示 11 月 11 号 11 点 11 分 11 秒执行一次（哈哈哈）。更多示例和讲解，可以看看 [《Spring Cron 表达式》](https://blog.csdn.net/bingduanlbd/article/details/51740913) 文章。注意，以调用**完成时刻**为开始计时时间。
- `fixedDelay` 属性：固定执行间隔，单位：毫秒。注意，以调用**完成时刻**为开始计时时间。
- `fixedRate` 属性：固定执行间隔，单位：毫秒。注意，以调用**开始时刻**为开始计时时间。
- 这三个属性，有点雷同，可以看看 [《@Scheduled 定时任务的fixedRate、fixedDelay、cron 的区别》](https://www.iteye.com/blog/guwq2014-2424520) ，一定要分清楚差异。

**不常用**属性如下：

- `initialDelay` 属性：初始化的定时任务执行延迟，单位：毫秒。
- `zone` 属性：解析 Spring Cron 表达式的所属的时区。默认情况下，使用服务器的本地时区。
- `initialDelayString` 属性：`initialDelay` 的字符串形式。
- `fixedDelayString` 属性：`fixedDelay` 的字符串形式。
- `fixedRateString` 属性：`fixedRate` 的字符串形式。

## 应用配置
在 application.yml 中，添加 Spring Task 定时任务的配置
```yml
spring:
  task:
    # Spring Task 调度任务的配置，对应 TaskSchedulingProperties 配置类
    scheduling:
      thread-name-prefix: pikaqiu-demo- # 线程池的线程名的前缀。默认为 scheduling- ，建议根据自己应用来设置
      pool:
        size: 10 # 线程池大小。默认为 1 ，根据自己应用来设置
      shutdown:
        await-termination: true # 应用关闭时，是否等待定时任务执行完成。默认为 false ，建议设置为 true
        await-termination-period: 60 # 等待任务完成的最大时长，单位为秒。默认为 0 ，根据自己应用来设置
```
在 spring.task.scheduling 配置项，Spring Task 调度任务的配置，对应 TaskSchedulingProperties 配置类。
Spring Boot TaskSchedulingAutoConfiguration 自动化配置类，实现 Spring Task 的自动配置，创建 ThreadPoolTaskScheduler 基于线程池的任务调度器。本质上，ThreadPoolTaskScheduler 是基于 ScheduledExecutorService 的封装，增强在调度时间上的功能。
注意，spring.task.scheduling.shutdown 配置项，是为了实现 Spring Task 定时任务的优雅关闭。我们想象一下，如果定时任务在执行的过程中，如果应用开始关闭，把定时任务需要使用到的 Spring Bean 进行销毁，例如说数据库连接池，那么此时定时任务还在执行中，一旦需要访问数据库，可能会导致报错。

所以，通过配置 await-termination = true ，实现应用关闭时，等待定时任务执行完成。这样，应用在关闭的时，Spring 会优先等待 ThreadPoolTaskScheduler 执行完任务之后，再开始 Spring Bean 的销毁。
同时，又考虑到我们不可能无限等待定时任务全部执行结束，因此可以配置 await-termination-period = 60 ，等待任务完成的最大时长，单位为秒。具体设置多少的等待时长，可以根据自己应用的需要。