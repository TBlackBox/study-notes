1. 在pom.xml中引入如下配置

   ```xml
   <dependency>    
       <groupId>org.springframework.cloud</groupId>    
       <artifactId>spring-cloud-starter-openfeign</artifactId> 
   </dependency>
   ```

2. 将需要调用的微服务集中处理到接口中

   ```java
   @Component //这里可以不要  但必须保证@EnableFeignClients 能扫描到@FeignClient修饰的类
   @FeignClient(value = "CLOUD-PAYMENT-SERVICE")
   public interface PaymentFeignService {
   
       @GetMapping(value = "/payment/get/{id}")
       public CommonResult getPaymentById(@PathVariable("id") Long id);
   }
   ```

   

**说明：**

`@GetMapping(value = "/payment/get/{id}")`这个就是要调的路径，

在controller中或者其他地方直接调就可以了

```java
@RestController
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        //这里就是调用服务
       return paymentFeignService.getPaymentById(id);
    }
}
 
```



3. 在主启动类中开启配置

   ```java
   @SpringBootApplication
   @EnableFeignClients  //开启OpenFeign
   public class OrderFeignMain
   {
       public static void main(String[] args) {
               SpringApplication.run(OrderFeignMain.class, args);
       }
   }
   ```

   