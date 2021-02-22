1. 配置服务熔断，注解的方式，需要在请求的方法上加上下面配置

   ```java
   //=====服务熔断
       @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
               @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
               @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数
               @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期
               @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),// 失败率达到多少后跳闸
       })
       public String paymentCircuitBreaker(@PathVariable("id") Integer id)
       {
           if(id < 0)
           {
               throw new RuntimeException("******id 不能负数");
           }
           String serialNumber = IdUtil.simpleUUID();
   
           return Thread.currentThread().getName()+"\t"+"调用成功，流水号: " + serialNumber;
       }
       public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id)
       {
           return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " +id;
       }
   ```

   2. 在启动类上加上下面配置

      ```java
      @SpringBootApplication
      @EnableEurekaClient
      @EnableCircuitBreaker  //开启熔断
      public class PaymentHystrixMain8001
      {
          public static void main(String[] args) {
                  SpringApplication.run(PaymentHystrixMain8001.class, args);
          }
      }
      ```

      