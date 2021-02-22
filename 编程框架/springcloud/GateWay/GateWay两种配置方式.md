# 简介

Spring Cloud Gateway 支持两种不同的用法：

- 编码式
- properties、yml 配置



1. 基于配置的形式，也就是在yml中配置

   ```yaml
   spring:
     application:
       name: dcs-gateway
     cloud:
       gateway:
         discovery:
           locator:
             enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
         routes:
           - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
             #uri: http://localhost:8001          #匹配后提供服务的路由地址
             uri: lb://dcs-provider-payment #匹配后提供服务的路由地址
             predicates:
               - Path=/payment/version/**         # 断言，路径相匹配的进行路由
   
           - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
             #uri: http://localhost:8001          #匹配后提供服务的路由地址
             uri: lb://dcs-provider-payment #匹配后提供服务的路由地址
             predicates:
               - Path=/payment/port/**         # 断言，路径相匹配的进行路由
               #- After=2020-02-21T15:51:37.485+08:00[Asia/Shanghai]
               #- Cookie=username,zzyy
               #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
   
   
   ```

   详细配置可以参考官方文档，这里只配置了断言

2. 基于硬编码的配置

   ```java
   @Configuration
   public class GateWayConfig {
   
       @Bean
       public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
   
           RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
           			//定义的路由ID 保证唯一即可   //定义的匹配路径   //跳转的地址
           routes.route("foword",r -> r.path("/nba").uri("http://www.qq.com")).build();
           return routes.build();
       }
   
       //还可以分配在其他的路由
   }
   ```

   访问  `网关:端口/nba` 将跳转到`http://www.qq.com/nba`去，详细的配置可以参考官方文档

