
# 简介
这个配置是springboot2.X的配置，1.x有区别，这里都不讲述了。在Spring Boot的众多Starter POMs中有一个特殊的模块，它不同于其他模块那样大多用于开发业务功能或是连接一些其他外部资源。它完全是一个用于暴露自身信息的模块，所以很明显，它的主要作用是用于监控与管理，它就是：spring-boot-starter-actuator。

spring-boot-starter-actuator模块的实现对于实施微服务的中小团队来说，可以有效地减少监控系统在采集应用指标时的开发量。当然，它也并不是万能的，有时候我们也需要对其做一些简单的扩展来帮助我们实现自身系统个性化的监控需求。

1. 引入依赖文件
```xml
//需要这个  方便查看
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. 开启安全权限

* 单个暴露端点的方式
```yaml
# 启用端点 env
management:
  endpoint:
    env:
      enabled: true
 
# 暴露端点 env 配置多个,隔开
management:
    endpoints:
        web:
        exposure:
            include: env
```

+ 全部暴露
```yaml
management:
  endpoint:
    env:
      enabled: true
  endpoints:
    web:
      exposure:
        include: '*'
```

***注意在使用Http访问端点时，需要加上默认/actuator 前缀***

3. 访问
```
http://127.0.0.1/actuator
```

得到下面信息
```
{
    "_links": {
        "self": {
            "href": "http://127.0.0.1/actuator",
            "templated": false
        },
        "auditevents": {
            "href": "http://127.0.0.1/actuator/auditevents",
            "templated": false
        },
        "beans": {
            "href": "http://127.0.0.1/actuator/beans",
            "templated": false
        },
        "caches-cache": {
            "href": "http://127.0.0.1/actuator/caches/{cache}",
            "templated": true
        },
        "caches": {
            "href": "http://127.0.0.1/actuator/caches",
            "templated": false
        },
        "health": {
            "href": "http://127.0.0.1/actuator/health",
            "templated": false
        },
        "health-component": {
            "href": "http://127.0.0.1/actuator/health/{component}",
            "templated": true
        },
        "health-component-instance": {
            "href": "http://127.0.0.1/actuator/health/{component}/{instance}",
            "templated": true
        },
        "conditions": {
            "href": "http://127.0.0.1/actuator/conditions",
            "templated": false
        },
        "configprops": {
            "href": "http://127.0.0.1/actuator/configprops",
            "templated": false
        },
        "env": {
            "href": "http://127.0.0.1/actuator/env",
            "templated": false
        },
        "env-toMatch": {
            "href": "http://127.0.0.1/actuator/env/{toMatch}",
            "templated": true
        },
        "info": {
            "href": "http://127.0.0.1/actuator/info",
            "templated": false
        },
        "loggers": {
            "href": "http://127.0.0.1/actuator/loggers",
            "templated": false
        },
        "loggers-name": {
            "href": "http://127.0.0.1/actuator/loggers/{name}",
            "templated": true
        },
        "heapdump": {
            "href": "http://127.0.0.1/actuator/heapdump",
            "templated": false
        },
        "threaddump": {
            "href": "http://127.0.0.1/actuator/threaddump",
            "templated": false
        },
        "metrics": {
            "href": "http://127.0.0.1/actuator/metrics",
            "templated": false
        },
        "metrics-requiredMetricName": {
            "href": "http://127.0.0.1/actuator/metrics/{requiredMetricName}",
            "templated": true
        },
        "scheduledtasks": {
            "href": "http://127.0.0.1/actuator/scheduledtasks",
            "templated": false
        },
        "httptrace": {
            "href": "http://127.0.0.1/actuator/httptrace",
            "templated": false
        },
        "mappings": {
            "href": "http://127.0.0.1/actuator/mappings",
            "templated": false
        }
    }
}
```

4. 对路径简单的介绍

|端点名|解释|
|-------|---------|
|autoconfig |所有自动配置信息|
|auditevents |审计事件|
|beans |所有Bean的信息|
|configprops |所有配置属性|
|dump |线程状态信息|
|env |当前环境信息|
|health |应用健康状况|
|info |当前应用信息|
|metrics |应用的各项指标|
|mappings |应用@RequestMapping映射路径|
|shutdown |关闭当前应用（默认关闭）|
|trace |追踪信息（最新的http请求）|

5. 定制端点信息  
* – 定制端点一般通过endpoints+端点名+属性名来设置。 
* – 修改端点id
```
endpoints.beans.id=mybeans
```
* – 开启远程应用关闭功能
```
endpoints.shutdown.enabled=true
```
* – 关闭端点
```
endpoints.beans.enabled=false
```
* – 开启所需端点
```
endpoints.enabled=false
endpoints.beans.enabled=true
```
+ – 定制端点访问根路径
```
management.context-path=/manage
```
+ 关闭http端点
```
management.port=-1
```


# 自定义健康指示器
springboot提供很很多默认指示器，例如：redis,mongdb，如果需要自定义，安装下面的步骤编写指示器。

 1。 编写一个指示器 实现 HealthIndicator 接口    
 2。 指示器的名字 xxxxHealthIndicator  
 3。 加入容器中

 例子
 ```
 @Component
public class MyAppHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {

        //自定义的检查方法
        //Health.up().build()代表健康
        return Health.down().withDetail("msg","服务异常").build();
    }
}

 ```