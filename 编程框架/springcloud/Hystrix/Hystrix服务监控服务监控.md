# 简介

Hystrix-dashboard 是一款针对 Hystrix 进行实时监控的工具，通过 Hystrix Dashboard 我们可以在直观地看到各 Hystrix Command 的请求响应时间, 请求成功率等数据。





# 列子

1. 在pom加入配置

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-hystrix</artifactId>
   </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
    </dependency>
   ```

   2. 在主启动类中添加如下配置

      ```java
      @EnableHystrix  //表示当前程序开启数据监控
      @EnableHystrixDashboard //表示当前程序开启数据监控的可视化视图程序
      public class ConsumerApplication {
          public static void main(String[] args) {
              SpringApplication.run(ConsumerApplication.class, args);
          }
      }
      ```

      

查看当前服务的监控信息---json字串格式的数据



```cpp
http://服务ip:端口/hystrix.stream
```

使用视图模式查看



```cpp
http://服务ip:端口/hystrix
```





**注意**

如果报错，需要子啊监控的服务启动类中添加如下配置

```java
@SpringBootApplication
public class PaymentHystrixMain8001
{
    public static void main(String[] args) {
            SpringApplication.run(PaymentHystrixMain8001.class, args);
    }


    /**
     *此配置是为了服务监控而配置，与服务容错本身无关，springcloud升级后的坑
     *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
     *只要在自己的项目里配置上下面的servlet就可以了
     */
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```





## 如何监控   图

![image-20210218164654862](../../../\sources\springcloud\image-20210218164654862.png)

![image-20210218164614145](../../../\sources\springcloud\image-20210218164614145.png)

![](../\图片\Hystrix的58.png)

![](../图片\Hystrix的61.png)