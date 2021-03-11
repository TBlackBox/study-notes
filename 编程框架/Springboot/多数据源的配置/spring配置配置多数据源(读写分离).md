# spring配置配置多数据源(读写分离)

现在大型点的项目都是采用数据库读写分离的形式，或者使用都数据源，本篇将介绍springboot使用读数据源，这篇介绍的是将配置好的数据源写下项目里面，缺陷介绍不能动态的添加和更矮数据源，将在高级篇里面动态的添加和删除数据源。



## 使用框架

- springboot
- mybatis

## 使用的数据源

springboot默认的Hikari

如果使用druid,则有点小变化



### 实现方案

Spring内置了一个`AbstractRoutingDataSource`，它可以把多个数据源配置成一个Map，然后，根据不同的key返回不同的数据源。因为`AbstractRoutingDataSource`也是一个DataSource接口，因此，应用程序可以先设置好key， 访问数据库的代码就可以从`AbstractRoutingDataSource`拿到对应的一个真实的数据源，从而访问指定的数据库。



# 配置案列

前提是正确的配置mybatis,在进行下面的数据库读写分离或多数据源的配置，这里主要说读数据源的配置。

1. 引入依赖包

   ```xml
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-jdbc</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```



2. 修改yml配置文件

   这里简单演示，为了方便说明，这里只使用了两个 数据库

   ```yml
   spring:
     datasource:
       driver-class-name: com.mysql.cj.jdbc.Driver
       jdbc-url: jdbc:mysql://127.0.0.1:3306/db_test?useUnicode=true&characterEncoding=UTF-erverTimezone=GMT%2b8&allowMultiQueries=true
       username: db_test
       password: db_test
       filters: wall,mergeStat
   
   
     slave:
       driver-class-name: com.mysql.cj.jdbc.Driver
       jdbc-url: jdbc:mysql://127.0.0.1:3306/db_test2?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2b8&allowMultiQueries=true
       username: db_test2
       password: db_test2
       filters: wall,mergeStat
   ```

3. 需要在启动类上排除springboot自动配置的数据源

   ```java
   @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
   ```
   
   
   
4. 创建数据源路由类

   ```java
   public class RoutingDataSource extends AbstractRoutingDataSource {
   
       static final Logger log = LoggerFactory.getLogger(RoutingDataSource.class);
   
       @Override
       protected Object determineCurrentLookupKey() {
           String key = RoutingDataSourceContext.getDataSourcesKey();
           log.info("当前数据库key：" + key);
           return key;
       }
   }
   ```

5. 新建一个枚举类，用于管理数据源

   ```java
   public enum DataSourceEnum {
   
       //默认数据源
       MASTER,
   
       SLAVE
   }
   ```

   

6. 将配置文件的数据源注入到spring容器中

   ```java
   @Configuration
   public class DataSourceConfig {
   
       private static final Logger log = LoggerFactory.getLogger(DataSourceConfig.class);
   
       @Bean("master")
       @ConfigurationProperties(prefix = "spring.datasource")
       public DataSource masterDataSource(){
           log.info("创建master 数据源");
           return DataSourceBuilder.create().build();
       }
   
   
       @Bean("slave")
       @ConfigurationProperties(prefix = "spring.slave")
       public DataSource slaveDataSource(){
           log.info("创建 slave 数据源");
           return DataSourceBuilder.create().build();
       }
   
       @Bean("myRoutingDataSource")
       @Primary //当有多个相同类型的Bean时，优先使用@Primary注解的Bean
       DataSource primaryDataSource(
               @Autowired @Qualifier("master") DataSource masterDataSource,
               @Autowired @Qualifier("slave") DataSource slaveDataSource
       ) {
           log.info("创建数据源路由。。。。");
           Map<Object, Object> map = new HashMap<>();
           map.put(DataSourceEnum.MASTER.name(), masterDataSource);
           map.put(DataSourceEnum.SLAVE.name(), slaveDataSource);
   
           RoutingDataSource routing = new RoutingDataSource();
           routing.setTargetDataSources(map);
           //设置默认的数据源  即主数据源
           routing.setDefaultTargetDataSource(masterDataSource);
           return routing;
       }
   ```

7. 由于多数据源，不需要spring boot默认的配置，需要配置SqlSessionFactory

   ```java
   @Configuration
   public class MybatisConfig {
   
       @Bean
       public SqlSessionFactory sqlSessionFactory(@Qualifier("myRoutingDataSource") DataSource dataSource) throws Exception {
           SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
           sqlSessionFactoryBean.setDataSource(dataSource);
   
           /*mybaits 配置文件*/
           sqlSessionFactoryBean.setTypeAliasesPackage("cn.yongzhi.example.db.domain");
           Resource configResource = new ClassPathResource("mybatis-config.xml");
           sqlSessionFactoryBean.setConfigLocation(configResource);
   
           sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath*:mapping/*.xml"));
           return sqlSessionFactoryBean.getObject();
       }
   
       //这里用的数据源是前面路由数据源
       @Bean
       public PlatformTransactionManager platformTransactionManager(@Qualifier("myRoutingDataSource")DataSource dataSource) {
           return new DataSourceTransactionManager(dataSource);
       }
   }
   ```

   **mybatis 的配置按照自己的配置填写即可**

8. 当然，我们需要配置一个存数据源的key，更加servlet线程模型的特点，使用ThreadLocal来存储最好。

   ```java
   public class RoutingDataSourceContext implements AutoCloseable {
   
       static final ThreadLocal<DataSourceEnum> threadLocalDataSourceKey = new ThreadLocal<>();
   
       public static String getDataSourcesKey(){
           DataSourceEnum key = threadLocalDataSourceKey.get();
           return key == null ? DataSourceEnum.MASTER.name():key.name();
       }
   
       public static void setDataSourceKey(DataSourceEnum dateSourceEnum){
           threadLocalDataSourceKey.set(dateSourceEnum);
       }
   
       public RoutingDataSourceContext(DataSourceEnum key){
           this.threadLocalDataSourceKey.set(key);
       }
   
   
       public static void remove(){
           threadLocalDataSourceKey.remove();
       }
   
   
   
       @Override
       public void close() throws Exception {
           threadLocalDataSourceKey.remove();
       }
   }
   ```

9. 测试

   我么在controller或者service里面，通过代码进行数据源的切换测试

   ```java
    @GetMapping("/testMS")
       public Object testMS() throws Exception {
           try(RoutingDataSourceContext ctx = new RoutingDataSourceContext(DataSourceEnum.SLAVE)){
   
               User user = new User();
   
               user.setName("老李");
               userService.save(user);
   
               log.info(userService.selectById(1).toString());
           }
   
           return userService.selectById(1);
       }
   ```

   测试可以发现，从try里面的数据源将会是slave,外面的将会是master数据源。从而实现了数据源的切换。

   这种方式肯定对开发者不友好，我们可以通过注解的方式来实现。

   

   # 通过注解的方式实现

   还是上面的配置和代码，加上下面的注解即可。

   1. 新建注解

      ```java
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.METHOD)
      @Documented
      public @interface DataSourceSwitcher {
      
          /**
           * 默认数据源 主数据源
           */
          DataSourceEnum value() default DataSourceEnum.MASTER;
      
      }
      ```

   2. 新建aop

      ```java
      @Aspect
      @Component
      public class DynamicDataSourceAspect implements Ordered {
      
          private static final Logger log = LoggerFactory.getLogger(DynamicDataSourceAspect.class);
      
      
      
      //3    @Around("@annotation(cn.yongzhi.example.db.multiple.rw.DataSourceSwitcher)")
      //1
         @Around("@annotation(org.springframework.transaction.annotation.Transactional)")
          public Object setDynamicDataSource(ProceedingJoinPoint pjp) throws Throwable {
              try {
                  MethodSignature signature = (MethodSignature) pjp.getSignature();
                 
      //4            
                  DataSourceSwitcher dataSourceSwitcher = signature.getMethod().getAnnotation(DataSourceSwitcher.class);
      
                  if(dataSourceSwitcher.value() != DataSourceEnum.MASTER){
                      RoutingDataSourceContext.setDataSourceKey(dataSourceSwitcher.value());
                      log.info("数据源切换至：" + dataSourceSwitcher.value().name());
                  }
      //2
      //            Transactional transactional =signature.getMethod().getAnnotation(Transactional.class);
      //            if(transactional.readOnly()){
      //                RoutingDataSourceContext.setDataSourceKey(DataSourceEnum.SLAVE);
      //                log.info("数据源切换至只读数据库");
      //            }
      
                  return pjp.proceed();
              } finally {
                      RoutingDataSourceContext.remove();
              }
          }
      
      
          @Override
          public int getOrder() {
              return 1;
          }
      }
      ```

   3. 测试

      ```java
      @DataSourceSwitcher(DataSourceEnum.SLAVE)
      @GetMapping("/testMSAnotation")
      public Object testMSAnotation() throws Exception {
          //        String key = "masterDataSource";
          User user = new User();
      
          user.setName("老李");
          userService.save(user);
      
          log.info("在里面查询出来的数据：");
          log.info(userService.selectById(1).toString());
      
          return userService.selectById(1);
      }
      ```

      在需要改变数据源的方法上加上注解即可，这个是自定义注解的写法。但我们可以通过spring自带的注解`Transactional`来区分，如果这个注解里面的`readonly`为true,也可以走从数据库,需要把aop的3，4注释起，1，2注释解开，就不需要加自定义注解了。当然，这里的读数据库只有一个，可以有多个，通过轮询的方式获取，只需要改一下算法既可，这里的例子也是为了方便理解，还有很大的优化空间。

# 总结

### 使用限制

受Servlet线程模型的局限，动态数据源不能在一个请求内设定后再修改，也就是`@DataSourceSwitcher`不能嵌套。此外，`@DataSourceSwitcher和`@Transactional`混用时，要设定AOP的优先级。



## 加入@Transactional注解或@LcnTransaction注解后并不能实现切换数据源，



原因如下：

1.  加@Transactional注解的方法：
   在执行方法之前，会先进入事务的拦截器，去获取Connection，此时AbstractRoutingDataSource中determineTargetDataSource()方法会决定要使用哪个数据源，因为在determineCurrentLookupKey()方法中定义默认使用key为write的数据源，所以当前获取的Connection中的数据源为主数据源。
   Conncetion获取后，将其放入在全局变量resources中，resources的类型为ThreadLocal<Map<Object, Object>>，存入的数据格式为<RoutingDataSource对象， ConnectionHolder对象>，ConnectionHolder中有Connection对象。
   等到调用执行SQL语句的方法时，再去获取Connection，会发现resources中已经存在相同的key，直接将其对应的ConnectionHolder对象取出来，而不在重新获取新的连接对象。
   AbstractRoutingDataSource中determineTargetDataSource()方法在第一次获取Connection时才会执行，所以，在同一个事务中使用的Connection始终是第一次得到的对象，实现不了数据源的切换。

2.  加@LcnTransaction注解的方法：
   在service中的方法上加@LcnTransaction注解，该方法被调用时，在执行到第一个操作sql语句的方法时获取Connection对象，在此之前缓存中并没有当前groupId对应的对象，所以通过AbstractRoutingDataSource中determineTargetDataSource()方法获取数据源，从而得到Connection，并将其放入缓存中，数据格式为<分布式事务的groupId，<LcnConnectionProxy类的路径，LcnConnectionProxy对象>>，LcnConnectionProxy对象中有Connection对象。
   当前方法中之后再取得的连接对象，都是在缓存中取到的。


# 可能会遇到的错误

**1. jdbcUrl is required with driverClassName错误解决**

解决方法

原来的配置

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://81.71.31.163:3306/db_test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2b8&allowMultiQueries=true
    username: db_test
    password: db_test
    filters: wall,mergeStat


  slave:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/db_test2?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2b8&allowMultiQueries=true
    username: db_test2
    password: db_test2
    filters: wall,mergeStat
```

修改为:

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    jdbc-url: jdbc:mysql://81.71.31.163:3306/db_test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2b8&allowMultiQueries=true
    username: db_test
    password: db_test
    filters: wall,mergeStat


  slave:
    driver-class-name: com.mysql.cj.jdbc.Driver
    jdbc-url: jdbc:mysql://127.0.0.1:3306/db_test2?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2b8&allowMultiQueries=true
    username: db_test2
    password: db_test2
    filters: wall,mergeStat
```

spring.datasource.url 数据库的 JDBC URL。

spring.datasource.jdbc-url 用来重写自定义连接池

官方文档的解释是：

> 因为连接池的实际类型没有被公开，所以在您的自定义数据源的元数据中没有生成密钥，而且在IDE中没有完成(因为DataSource接口没有暴露属性)。另外，如果您碰巧在类路径上有Hikari，那么这个基本设置就不起作用了，因为Hikari没有url属性(但是确实有一个jdbcUrl属性)。在这种情况下，您必须重写您的配置如下:
（如果用druid话，那就不需要了）

