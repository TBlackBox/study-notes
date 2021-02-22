# 简介

当服务很多时,单靠代码手动管理是很麻烦的,需要一个公共组件,统一管理多服务,包括服务是否正常运行,等

Eureka用于**==服务注册==**,目前官网**已经停止更新**，这里简单得介绍，如果要详细得学习 还需要到官网或者github上查看文档







Eureka 包含两个组件

- Eureka Service： 通过服务注册服务

  各个服务节点通过配置启动后，会在Eureka Server中心进行注册，这样EurekaServer中的服务注册表中将会存储所有可用的服务节点信息，服务节点信息可以在界面上直观看到。

- Eureka Client：通过注册中心进行访问

是一个java客服端，用于简化Eureka Server的交互，客服端同时也具备一个内置的，使用轮询负载算法的负载均衡器，在应用启动后，将会想EurekaServer 发送心跳（默认周期为30s),如果EurekaServer 在多个心跳周期内没有接受到某个节点的心跳，eurekaServer 将会从服务注册表中把这个服务节点移除（默认90秒）。

# 配置

注意：这里的配置事列是eureka2.0

## 单价版配置

### 服务端配置

1. 创建springboot 项目，在pom.xml添加如下配置

   ````xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   </dependency>
   ````

2.  在yml 配置文件中添加如下配置

   ```yml
   eureka:
     instance:
       hostname: eureka7001.com #eureka服务端的实例名称
     client:
       register-with-eureka: false     #false表示不向注册中心注册自己。
       fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
       service-url:
       #集群指向其它eureka
         #defaultZone: http://eureka7002.com:7002/eureka/
       #单机就是7001自己
         defaultZone: http://eureka7001.com:7001/eureka/
     #server:
       #关闭自我保护机制，保证不可用服务被及时踢除
       #enable-self-preservation: false
       #eviction-interval-timer-in-ms: 2000
   ```

3. 在启动类开启eureka 服务端

   ```java
   @SpringBootApplication
   @EnableEurekaServer //开启eureka 服务端注解
   public class EurekaMain7001
   {
       public static void main(String[] args) {
               SpringApplication.run(EurekaMain7001.class, args);
       }
   }
   ```

   4. 严重服务端配置是否成功,访问如下地址

      ```
      http://127.0.0.1:7001/
      ```

      如果出现了spring eureka 的界面，说明配置成功



## 客服端配置

1. 在需要注册的微服务的pom.xml添加如下配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2. 修改配置yml 配置文件

   ```yml
   eureka:
     client:
       #表示是否将自己注册进EurekaServer默认为true。
       register-with-eureka: true
       #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
       fetchRegistry: true
       service-url:
         #单机版
         defaultZone: http://localhost:7001/eureka
         # 集群版
         #defaultZone: http://地址1:7001/eureka,http://地址2:7002/eureka
     instance:
       instance-id: payment
       #访问路径可以显示IP地址
       prefer-ip-address: true
       #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
       #lease-renewal-interval-in-seconds: 1
       #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
       #lease-expiration-duration-in-seconds: 2
   ```

   3. 在需要注册的服务启动类上添加注解，启动eureka 客服端

      ```java
      @SpringBootApplication
      @EnableEurekaClient  //启动eureka 客服端
      public class DcsProviderPaymentApplication {
      
          public static void main(String[] args) {
              SpringApplication.run(DcsProviderPaymentApplication.class, args);
          }
      
      }
      ```

      4. 严重是否注册成功

         刷新上面验证服务端是否安装成功页面，在`DS Replicas`会出现上面的服务名称，我们在其他服务中，通过服务名称，就可以访问微服务了。 



## 集群版

eureka集群版是相互注册，相互守望，把单机版配置好后，在eureka 服务端的yml文件配置文件中添加其他eureka的服务地址即可,单价版都只有他自己

```yml
defaultZone : eureka服务器地址1,eueka服务器地址2。。。。
```

客服端同样修改yml 配置

```
defaultZone : eureka服务器地址1,eueka服务器地址2。。。。
```



# 获取服务的注册信息

我们可以在一个注册的微服务中，获取微服务的注册信息,注入`DiscoveryClient`

```java
@Resource
private DiscoveryClient discoveryClient;
```

获取信息

```java
 @GetMapping(value = "/payment/discovery")
public Object discovery()
{
    List<String> services = discoveryClient.getServices();
    for (String element : services) {
        log.info("*****element: "+element);
    }

    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
    for (ServiceInstance instance : instances) {
        log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
    }

    return this.discoveryClient;
}
```



# Eureka自我保护

我们配置完eureka 服务端后，在验证的页面，有这样一句话

```
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
DS Replicas
```

保护模式主要用于一组客户端和Eureka Serer之间存在的网络分区场景下的保护，一旦进入保护模式***eureka Server 将会尝试保护其服务注册表中的信息，不在删除服务注册表中的数据，也不会注销任何微服务***



可以这样理解：

某时刻某一个微服务不可用了，eureka不会立即清理，依旧会对微服务信息进行保存，这输入CAP里面的AP分支。可以通过配置的一些设置开启或者关闭。