## 注解原型
```java
@Target({ElementType.TYPE})   //表示作用在接口上
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface FeignClient {
    @AliasFor("name")
    String value() default "";    //指定FeignClient的名称，如果项目使用了Ribbon，name属性会作为微服务的名称，用于服务发现

    /** @deprecated */
    @Deprecated
    String serviceId() default "";

    String contextId() default "";

    @AliasFor("value")
    String name() default "";  //同value

    String qualifier() default "";

    String url() default "";  //url一般用于调试，可以手动指定@FeignClient调用的地址 例如调用不是微服务的客户端

    boolean decode404() default false;  //当发生http 404错误时，如果该字段位true，会调用decoder进行解码，否则抛出FeignException

    Class<?>[] configuration() default {}; //Feign配置类，可以自定义Feign的Encoder、Decoder、LogLevel、Contract

    Class<?> fallback() default void.class; //定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口

    Class<?> fallbackFactory() default void.class; //工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码

    String path() default ""; //定义当前FeignClient的统一前缀

    boolean primary() default true;
}

```

在使用fallback属性时，需要使用@Component注解，保证fallback类被Spring容器扫描到.

```java
@FeignClient(value = "socket",fallback = PublishClient.TestFallback.class)
public interface PublishClient {

    @GetMapping(value = "/socket/test",consumes = MediaType.APPLICATION_JSON_VALUE)
    @ResponseBody
    CommonResult test();

    @Component
    class TestFallback implements PublishClient{

        @Override
        public CommonResult test() {
            return null;
        }
    }
}
```

在使用FeignClient时，Spring会按name创建不同的ApplicationContext，通过不同的Context来隔离FeignClient的配置信息，在使用配置类时，不能把配置类放到Spring App Component scan的路径下，否则，配置类会对所有FeignClient生效.

```java
@FeignClient(value = "socket",fallback = PublishClient.TestFallback.class,configuration = FeignConfig.class)
public interface PublishClient {

    @GetMapping(value = "/socket/test",consumes = MediaType.APPLICATION_JSON_VALUE)
    @ResponseBody
    CommonResult test();

    @Component
    class TestFallback implements PublishClient{

        @Override
        public CommonResult test() {
            return null;
        }
    }
}

//这样的话  所有的feign 都会用到
@Configuration
@EnableFeignClients(basePackages = {"com.cqaif7.*"})
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel()
    {
        return Logger.Level.FULL;  //这里返回日志级别
    }
}

```

