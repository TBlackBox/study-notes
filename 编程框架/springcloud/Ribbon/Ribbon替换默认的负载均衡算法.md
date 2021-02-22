1. 特别注意，定义的配置类必须放到`@ComponentScan`扫描不到的包中，也就是`@SpringBootApplication`注解所在的包之外包含子包

2. 添加如下配置类即可

   ```java
   Configuration
   public class MySelfRule {
   
       @Bean
       public IRule myRule(){
           return new RandomRule();//定义为随机
       }
   }
   ```

   3. 在启动类引入配置

      ```java
      @EnableEurekaClient
      @SpringBootApplication
      @RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
      public class OrderMain80 {
          public static void main(String[] args) {
              SpringApplication.run(OrderMain80.class,args);
          }
      }
       
      ```

      

## 手写ribbon负载均衡算法

可以去看springclound 文档