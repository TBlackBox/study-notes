# 配置方法1

1. 映入jar包(hystrix配置一般实在客户端)

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   ```

   2. 配置yal

   ```yaml
   feign:
     hystrix:
       enabled: true
   ```

   

3. 配置事列

   - 单独的配置例子

   ```java
   import com.atguigu.springcloud.service.PaymentHystrixService;
   import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   @DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
   public class OrderHystirxController
   {
       @Resource
       private PaymentHystrixService paymentHystrixService;
   
   
       @GetMapping("/consumer/payment/hystrix/timeout/{id}")
       @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
               @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
       })
       public String paymentInfo_TimeOut(@PathVariable("id") Integer id)
       {
           //int age = 10/0;  //超时或者异常都会导致调用paymentTimeOutFallbackMethod方法
           String result = paymentHystrixService.paymentInfo_TimeOut(id);
           return result;
       }
       
       public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id)
       {
           return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
       }
   }
   
   ```

   - 全局的配置列子

     ```java
     import com.atguigu.springcloud.service.PaymentHystrixService;
     import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
     import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
     import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
     import lombok.extern.slf4j.Slf4j;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.PathVariable;
     import org.springframework.web.bind.annotation.RestController;
     
     import javax.annotation.Resource;
     
     @RestController
     @Slf4j
     @DefaultProperties(defaultFallback = "payment_Global_FallbackMethod") //使用全局的
     public class OrderHystirxController
     {
         @Resource
         private PaymentHystrixService paymentHystrixService;
     
         @GetMapping("/consumer/payment/hystrix/timeout/{id}")
         @HystrixCommand //不指定  就用全局的
         public String paymentInfo_TimeOut(@PathVariable("id") Integer id)
         {
             int age = 10/0;
             String result = paymentHystrixService.paymentInfo_TimeOut(id);
             return result;
         }
     
         // 下面是全局fallback方法
         public String payment_Global_FallbackMethod()
         {
             return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
         }
     }
     
     ```

     4. 在主启动类上开启hystrix

        ```java
        @SpringBootApplication
        @EnableFeignClients
        @EnableHystrix
        public class OrderHystrixMain80
        {
            public static void main(String[] args)
            {
                SpringApplication.run(OrderHystrixMain80.class,args);
            }
        }
        
        ```

        问题：

        每个方法配置一个？？？膨胀

        和业务逻辑混一起？？？混乱

        

        # 配置方法2

        上面的配置比较傻x,虽然技术上可以实现，但是不友好，代码臃肿并且不好维护，下面进行统一处理配置

        1. 有一个Feign的接口用于调用远程服务

           ```java
           @Component
           @FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT" ,fallback = PaymentFallbackService.class) //这里配置实现的接口实现类
           public interface PaymentHystrixService
           {
               @GetMapping("/payment/hystrix/ok/{id}")
               public String paymentInfo_OK(@PathVariable("id") Integer id);
           
               @GetMapping("/payment/hystrix/timeout/{id}")
               public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
           }
           ```

           2. 实现这个接口，并进行相应的配置即可

              ```java
              @Component
              public class PaymentFallbackService implements PaymentHystrixService
              {
                  @Override
                  public String paymentInfo_OK(Integer id)
                  {
                      return "-----PaymentFallbackService fall back-paymentInfo_OK ,o(╥﹏╥)o";
                  }
              
                  @Override
                  public String paymentInfo_TimeOut(Integer id)
                  {
                      return "-----PaymentFallbackService fall back-paymentInfo_TimeOut ,o(╥﹏╥)o";
                  }
              }
              ```

              这样的话，就统一的进行了配置和管理

###### 它的运行逻辑是
​		当请求过来,首先还是通过Feign远程调用pay模块对应的方法
​    但是如果pay模块报错,调用失败,那么就会调用PayMentFalbackService类的
​    当前同名的方法,作为降级方法

**这样虽然解决了代码耦合度问题,但是又出现了过多重复代码的问题,每个方法都有一个降级方法**