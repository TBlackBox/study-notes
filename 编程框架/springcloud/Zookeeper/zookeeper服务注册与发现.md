1. 启动zookeeper,需要在linux 上面安装

2. 配置服务上配置yml

   ```yaml
   #8004表示注册到zookeeper服务器的支付服务提供者端口号
   server:
     port: 8004
   
   
   #服务别名----注册zookeeper到注册中心名称
   spring:
     application:
       name: 服务别名
     cloud:
       zookeeper:
         connect-string: zookeeper服务器地址:端口
   ```

3. 配置主启动类

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
   public class PaymentMain8004
   {
       public static void main(String[] args) {
               SpringApplication.run(PaymentMain8004.class, args);
       }
   }
   
   ```



4. 需要注意版本问题，根据自己安装的zookeeper服务端版本引入zookeeper 客服端版本

```xml
<!-- SpringBoot整合zookeeper客户端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <!--先排除自带的zookeeper3.5.3-->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--添加zookeeper3.4.9版本-->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
</dependency>
```



注意：我们在zk上注册的node是临时节点,当我们的服务一定时间内没有发送心跳
  	那么zk就会`将这个服务的node删除了