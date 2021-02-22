1. 新建工程，引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-gateway</artifactId>
   </dependency>
   
   ```



2. 修改yml 配置

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

3. 启动即可，上面的配置使用了eureka 注册中心，从注册中心动态获取配置。也可以使用代码配置的方式



## 报错解决

1. 可能出现下面的错误

```
Spring MVC found on classpath, which is incompatible with Spring Cloud Gateway at this time. Please remove spring-boot-starter-web dependency.
```

解决办法：

移除`spring-boot-starter-web`的依赖



原因：

由于GateWay 集成了Spring WebFlux,这里强制要求去除spring-boot-starter-web

