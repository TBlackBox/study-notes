# OpenFeign日志

Feign提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解Feign 中http请求的细节。说白了就是对feign接口的调用情况进行监控和输出。  



## OpenFeign的日志级别

- NONE:默认的，不显示任何日志；
- BASIC:仅记录请求方法，URL,响应状态码及执行时间；
- HEADERS：除了BASIC中定义的信息之外，还有请求和响应头信息；
- FULL:除了HEADERS中定义的信息之外，还有请求和响应正文及元数据；





##  配置日志级别

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig
{
    @Bean
    Logger.Level feignLoggerLevel()
    {
        return Logger.Level.FULL;  //这里返回日志级别
    }
}
```



## 为指定类设置日志级别

在yml中配置

```yaml
logging:
  level:
    com.PaymentFeignService: debug  #com.PaymentFeignService是接口路径或者类路径
```

