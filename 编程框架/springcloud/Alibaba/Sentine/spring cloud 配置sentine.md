# 控制台安装与配置

1. 下载地址

   ```
   https://github.com/alibaba/Sentinel/releases
   ```

   需要的环境，java8环境OK，并且8080端口不能被占用

2. 启动命令

   ```
   java -jar sentinel-dashboard-版本号.jar
   ```

3. 控制台访问命令

   ```
   sentinel 默认端口是8080
   http://localhost:8080
   默认账号和密码都是
   sentinel
   sentinel
   ```
   
# 客服端安装与配置

1. 引入依赖
```xml

    <!--SpringCloud ailibaba nacos -->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
   <dependency>
       <groupId>com.alibaba.csp</groupId>
       <artifactId>sentinel-datasource-nacos</artifactId>
   </dependency>
   <!--SpringCloud ailibaba sentinel -->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   <!--openfeign-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
```

2. 修改yml配置

```yml
   spring:
     application:
       name: dcs-sentinel-service
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848 #Nacos服务注册中心地址
       sentinel:
         transport:
           dashboard: 127.0.0.1:8080 #配置Sentinel dashboard地址
           #默认端口是8719，假如被占用会自动从8719开始一吃依次加1，直到找到未被占用的端口为止
           port: 8719
         #一旦我们重启应用，Sentinel规则将消失，生产环境需要将配置规则进行持久化,这里的配置是为了将sentinel的配置持久化到nacos里面，意思也就是用来sentinel 就得用nacos,不然限流配置这些无法持久化
         datasource:
           ds1:
             nacos:
               server-addr: localhost:8848
               dataId: dcs-sentinel-service
               groupId: DEFAULT_GROUP
               data-type: json
               rule-type: flow
   
   management:
     endpoints:
       web:
         exposure:
           include: '*'
   # 这点要开启feign 对sentinel的支持
   feign:
     sentinel:
       enabled: true # 激活Sentinel对Feign的支持
   
```
这里只是一个简简单的步骤，详细的得去都官方得文档，进而灵活运用。