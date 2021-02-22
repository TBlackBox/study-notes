1. 引入consul包

   ```xml
    <!--SpringCloud consul-server -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-consul-discovery</artifactId>
   </dependency>
   ```

   

2. 修改yml

   ```yaml
   ###consul服务端口号
   server:
     port: 8006
   
   spring:
     application:
       name: consul-provider-payment
   ####consul注册中心地址
     cloud:
       consul:
         host: localhost
         port: 8500
         discovery:
           #hostname: 127.0.0.1
           service-name: ${spring.application.name}
   
   
   ```

3. 在主启动上加入启动注解

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class PaymentMain8006
   {
       public static void main(String[] args) {
               SpringApplication.run(PaymentMain8006.class, args);
       }
   }
   ```

   

