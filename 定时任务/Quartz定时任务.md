# 简介

​        Quartz是一个开源的作业调度框架，它完全由Java写成，并设计用于J2SE和J2EE应用中。它提供了巨大的灵 活性而不牺牲简单性。你能够用它来为执行一个作业而创建简单的或复杂的调度。它有很多特征，如：数据库支持，集群，插件，EJB作业预构 建，JavaMail及其它，支持cron-like表达式等等（瓶颈应该也是由于数据库造成的）。

​        Quartz 自带了集群方案。它通过将作业信息存储到关系数据库中，并使用关系数据库的**行锁**来实现执行作业的竞争，从而保证多个进程下，同一个任务在相同时刻，不能重复执行。



在 Quartz 体系结构中，有三个组件非常重要：

- Scheduler ：调度器
- Trigger ：触发器
- Job ：任务



# 体系结构

- Job接口
```java
public interface Job {
    void execute(JobExecutionContext context)
        throws JobExecutionException;
}
```
只定义一个方法execute(JobExecutionContext context)，在实现接口的execute方法中编写所需要定时执行的Job(任务)， JobExecutionContext类提供了调度应用的一些信息。Job运行时的信息保存在JobDataMap实例中；

- JobDetail
 Quartz每次调度Job时， 都重新创建一个Job实例， 所以它不直接接受一个Job的实例，相反它接收一个Job实现类(JobDetail:描述Job的实现类及其它相关的静态信息，如Job名字、描述、关联监听器等信息)，以便运行时通过newInstance()的反射机制实例化Job。

- Trigger
描述触发Job执行的时间触发规则。主要有SimpleTrigger和CronTrigger这两个子类。当且仅当需调度一次或者以固定时间间隔周期执行调度，SimpleTrigger是最适合的选择；而CronTrigger则可以通过Cron表达式定义出各种复杂时间规则的调度方案：如工作日周一到周五的15：00~16：00执行调度等；

- Calendar
org.quartz.Calendar和java.util.Calendar不同， 它是一些日历特定时间点的集合（可以简单地将org.quartz.Calendar看作java.util.Calendar的集合——java.util.Calendar代表一个日历时间点，无特殊说明后面的Calendar即指org.quartz.Calendar）。 一个Trigger可以和多个Calendar关联， 以便排除或包含某些时间点。假设，我们安排每周星期一早上10:00执行任务，但是如果碰到法定的节日，任务则不执行，这时就需要在Trigger触发机制的基础上使用Calendar进行定点排除。针对不同时间段类型，Quartz在org.quartz.impl.calendar包下提供了若干个Calendar的实现类，如AnnualCalendar、MonthlyCalendar、WeeklyCalendar分别针对每年、每月和每周进行定义；

- Scheduler
代表一个Quartz的独立运行容器， Trigger和JobDetail可以注册到Scheduler中， 两者在Scheduler中拥有各自的组及名称， 组及名称是Scheduler查找定位容器中某一对象的依据， Trigger的组及名称必须唯一， JobDetail的组和名称也必须唯一（但可以和Trigger的组和名称相同，因为它们是不同类型的）。Scheduler定义了多个接口方法， 允许外部通过组及名称访问和控制容器中Trigger和JobDetail。Scheduler可以将Trigger绑定到某一JobDetail中， 这样当Trigger触发时， 对应的Job就被执行。一个Job可以对应多个Trigger， 但一个Trigger只能对应一个Job。可以通过SchedulerFactory创建一个Scheduler实例。Scheduler拥有一个SchedulerContext，它类似于ServletContext，保存着Scheduler上下文信息，Job和Trigger都可以访问SchedulerContext内的信息。SchedulerContext内部通过一个Map，以键值对的方式维护这些上下文数据，SchedulerContext为保存和获取数据提供了多个put()和getXxx()的方法。可以通过Scheduler# getContext()获取对应的SchedulerContext实例；

- ThreadPool
Scheduler使用一个线程池作为任务运行的基础设施，任务通过共享线程池中的线程提高运行效率。Job有一个StatefulJob子接口，代表有状态的任务，该接口是一个没有方法的标签接口，其目的是让Quartz知道任务的类型，以便采用不同的执行方案。无状态任务在执行时拥有自己的JobDataMap拷贝，对JobDataMap的更改不会影响下次的执行。而有状态任务共享共享同一个JobDataMap实例，每次任务执行对JobDataMap所做的更改会保存下来，后面的执行可以看到这个更改，也即每次执行任务后都会对后面的执行发生影响。正因为这个原因，无状态的Job可以并发执行，而有状态的StatefulJob不能并发执行，这意味着如果前次的StatefulJob还没有执行完毕，下一次的任务将阻塞等待，直到前次任务执行完毕。有状态任务比无状态任务需要考虑更多的因素，程序往往拥有更高的复杂度，因此除非必要，应该尽量使用无状态的Job。如果Quartz使用了数据库持久化任务调度信息，无状态的JobDataMap仅会在Scheduler注册任务时保持一次，而有状态任务对应的JobDataMap在每次执行任务后都会进行保存。Trigger自身也可以拥有一个JobDataMap，其关联的Job可以通过JobExecutionContext#getTrigger().getJobDataMap()获取Trigger中的JobDataMap。不管是有状态还是无状态的任务，在任务执行期间对Trigger的JobDataMap所做的更改都不会进行持久，也即不会对下次的执行产生影响。Quartz拥有完善的事件和监听体系，大部分组件都拥有事件，如任务执行前事件、任务执行后事件、触发器触发前事件、触发后事件、调度器开始事件、关闭事件等等，可以注册相应的监听器处理感兴趣的事件。

# 简单的使用方式
1. 定义任务
```java
public class HelloQuartz  implements Job {  
   
    public void execute(JobExecutionContext arg0) throws JobExecutionException {  
        System.out.println("Hello Quartz !");                 
    }         
} 
```
2. 启动
```java 
 public static void main(String[] args) throws InterruptedException {    
       
       //通过schedulerFactory获取一个调度器    
       SchedulerFactory schedulerfactory = new StdSchedulerFactory();    
       Scheduler scheduler=null;    
       try{    
           // 通过schedulerFactory获取一个调度器    
           scheduler = schedulerfactory.getScheduler();    
               
            // 创建jobDetail实例，绑定Job实现类    
            // 指明job的名称，所在组的名称，以及绑定job类    
           JobDetail job = JobBuilder.newJob(HelloQuartz.class).withIdentity("JobName", "JobGroupName").build();    
             
               
            // 定义调度触发规则    
                           
            // SimpleTrigger   
//      Trigger trigger=TriggerBuilder.newTrigger().withIdentity("SimpleTrigger", "SimpleTriggerGroup")    
//                    .withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(3).withRepeatCount(6))    
//                    .startNow().build();    
             
            //  corn表达式  每五秒执行一次  
              Trigger trigger=TriggerBuilder.newTrigger().withIdentity("CronTrigger1", "CronTriggerGroup")    
              .withSchedule(CronScheduleBuilder.cronSchedule("*/5 * * * * ?"))    
              .startNow().build();     
               
            // 把作业和触发器注册到任务调度中    
           scheduler.scheduleJob(job, trigger);    
               
           // 启动调度    
           scheduler.start();    
             
           Thread.sleep(10000);  
             
           // 停止调度  
           scheduler.shutdown();  
               
       }catch(SchedulerException e){    
           e.printStackTrace();    
       }    
           
   } 
```


# Quartz 单机
1. 引入依赖
```xml
<!-- 实现对 Quartz 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```
2. 创建任务类
```java
public class DemoJob01 extends QuartzJobBean {

    private Logger logger = LoggerFactory.getLogger(getClass());

    private final AtomicInteger counts = new AtomicInteger();

    @Autowired
    private DemoService demoService;

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        logger.info("[executeInternal][定时第 ({}) 次执行, demoService 为 ({})]", counts.incrementAndGet(),
                demoService);
    }

}
```

继承 QuartzJobBean 抽象类，实现 #executeInternal(JobExecutionContext context) 方法，执行自定义的定时任务的逻辑。

QuartzJobBean 实现了 org.quartz.Job 接口，提供了 Quartz 每次创建 Job 执行定时逻辑时，将该 Job Bean 的依赖属性注入。例如说，DemoJob01 需要 @Autowired 注入的 demoService 属性。核心代码如下：

// QuartzJobBean.java

public final void execute(JobExecutionContext context) throws JobExecutionException {
    try {
        // 将当前对象，包装成 BeanWrapper 对象
        BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
        // 设置属性到 bw 中
        MutablePropertyValues pvs = new MutablePropertyValues();
        pvs.addPropertyValues(context.getScheduler().getContext());
        pvs.addPropertyValues(context.getMergedJobDataMap());
        bw.setPropertyValues(pvs, true);
	} catch (SchedulerException ex) {
		throw new JobExecutionException(ex);
	}

    // 执行提供给子类实现的抽象方法
    this.executeInternal(context);
}

protected abstract void executeInternal(JobExecutionContext context) throws JobExecutionException;
这样一看，是不是清晰很多。不要惧怕中间件的源码，好奇哪个类或者方法，就点进去看看。反正，又不花钱。
counts 属性，计数器。用于我们后面我们展示，每次 DemoJob01 都会被 Quartz 创建出一个新的 Job 对象，执行任务。这个很重要，也要非常小心。

创建 DemoJob02 类，示例定时任务 02 类。代码如下：

// DemoJob02.java

public class DemoJob02 extends QuartzJobBean {

    private Logger logger = LoggerFactory.getLogger(getClass());
    
    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        logger.info("[executeInternal][我开始的执行了]");
    }

}
比较简单，为了后面演示案例之用。

3. 配置
```java
@Configuration
public class ScheduleConfiguration {

    public static class DemoJob01Configuration {

        @Bean
        public JobDetail demoJob01() {
            return JobBuilder.newJob(DemoJob01.class)
                    .withIdentity("demoJob01") // 名字为 demoJob01
                    .storeDurably() // 没有 Trigger 关联的时候任务是否被保留。因为创建 JobDetail 时，还没 Trigger 指向它，所以需要设置为 true ，表示保留。
                    .build();
        }

        @Bean
        public Trigger demoJob01Trigger() {
            // 简单的调度计划的构造器
            SimpleScheduleBuilder scheduleBuilder = SimpleScheduleBuilder.simpleSchedule()
                    .withIntervalInSeconds(5) // 频率。
                    .repeatForever(); // 次数。
            // Trigger 构造器
            return TriggerBuilder.newTrigger()
                    .forJob(demoJob01()) // 对应 Job 为 demoJob01
                    .withIdentity("demoJob01Trigger") // 名字为 demoJob01Trigger
                    .withSchedule(scheduleBuilder) // 对应 Schedule 为 scheduleBuilder
                    .build();
        }

    }

    public static class DemoJob02Configuration {

        @Bean
        public JobDetail demoJob02() {
            return JobBuilder.newJob(DemoJob02.class)
                    .withIdentity("demoJob02") // 名字为 demoJob02
                    .storeDurably() // 没有 Trigger 关联的时候任务是否被保留。因为创建 JobDetail 时，还没 Trigger 指向它，所以需要设置为 true ，表示保留。
                    .build();
        }

        @Bean
        public Trigger demoJob02Trigger() {
            // 基于 Quartz Cron 表达式的调度计划的构造器
            CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("0/10 * * * * ? *");
            // Trigger 构造器
            return TriggerBuilder.newTrigger()
                    .forJob(demoJob02()) // 对应 Job 为 demoJob02
                    .withIdentity("demoJob02Trigger") // 名字为 demoJob02Trigger
                    .withSchedule(scheduleBuilder) // 对应 Schedule 为 scheduleBuilder
                    .build();
        }

    }

}
```

- 内部创建了 DemoJob01Configuration 和 DemoJob02Configuration 两个配置类，分别配置 DemoJob01 和 DemoJob02 两个 Quartz Job 。
- ========== **DemoJob01Configuration** ==========
- `#demoJob01()` 方法，创建 DemoJob01 的 JobDetail Bean 对象。
- `#demoJob01Trigger()` 方法，创建 DemoJob01 的 Trigger Bean 对象。其中，我们使用 [SimpleScheduleBuilder](https://github.com/quartz-scheduler/quartz/blob/master/quartz-core/src/main/java/org/quartz/SimpleScheduleBuilder.java) 简单的调度计划的构造器，创建了每 5 秒执行一次，无限重复的调度计划。
- ========== **DemoJob2Configuration** ==========
- `#demoJob2()` 方法，创建 DemoJob02 的 JobDetail Bean 对象。
- `#demoJob02Trigger()` 方法，创建 DemoJob02 的 Trigger Bean 对象。其中，我们使用 [CronScheduleBuilder](https://github.com/quartz-scheduler/quartz/blob/master/quartz-core/src/main/java/org/quartz/CronScheduleBuilder.java) 基于 Quartz Cron 表达式的调度计划的构造器，创建了每**第** 10 秒执行一次的调度计划。这里，推荐一个 [Quartz/Cron/Crontab 表达式在线生成工具](http://www.bejson.com/othertools/cron/) ，方便帮我们生成 Quartz Cron 表达式，并计算出最近 5 次运行时间。

😈 因为 JobDetail 和 Trigger 一般是成双成对出现，所以艿艿习惯配置成一个 Configuration 配置类。

4. 启动springboot 项目即可

   - 项目启动时，会创建了 Quartz QuartzScheduler 并启动。
   - 考虑到阅读日志方便，艿艿这里把 DemoJob01 和 DemoJob02 的日志分开来了。
   - 对于 DemoJob01 ，每 5 秒左右执行一次。同时我们可以看到，`demoService` 成功注入，而 `counts` 每次都是 1 ，说明每次 DemoJob01 都是新创建的。
   - 对于 DemoJob02 ，每**第** 10 秒执行一次。

5. 配置

   ```yml
   应用配置文件
   在 application.yml 中，添加 Quartz 的配置，如下：
   
   spring:
     # Quartz 的配置，对应 QuartzProperties 配置类
     quartz:
       job-store-type: memory # Job 存储器类型。默认为 memory 表示内存，可选 jdbc 使用数据库。
       auto-startup: true # Quartz 是否自动启动
       startup-delay: 0 # 延迟 N 秒启动
       wait-for-jobs-to-complete-on-shutdown: true # 应用关闭时，是否等待定时任务执行完成。默认为 false ，建议设置为 true
       overwrite-existing-jobs: false # 是否覆盖已有 Job 的配置
       properties: # 添加 Quartz Scheduler 附加属性，更多可以看 http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/configuration.html 文档
         org:
           quartz:
             threadPool:
               threadCount: 25 # 线程池大小。默认为 10 。
               threadPriority: 5 # 线程优先级
               class: org.quartz.simpl.SimpleThreadPool # 线程池类型
   #    jdbc: # 这里暂时不说明，使用 JDBC 的 JobStore 的时候，才需要配置
   
   ```

   - 在 `spring.quartz` 配置项，Quartz 的配置，对应 [QuartzProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/quartz/QuartzProperties.java) 配置类。
   - Spring Boot [QuartzAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/quartz/QuartzAutoConfiguration.java) 自动化配置类，实现 Quartz 的自动配置，创建 Quartz [Scheduler](https://github.com/quartz-scheduler/quartz/blob/master/quartz-core/src/main/java/org/quartz/Scheduler.java)(**调度器**) Bean 。

   **注意**，`spring.quartz.wait-for-jobs-to-complete-on-shutdown` 配置项，是为了实现 Quartz 的优雅关闭，建议开启。关于这块，和我们在 Spring Task 的[「2.6 应用配置文件」](https://www.iocoder.cn/Spring-Boot/Job/?yudao#) 提到的是一致的。
