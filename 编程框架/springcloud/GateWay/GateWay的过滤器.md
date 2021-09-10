# Filter

Spring Cloud Gateway 中的过滤器分为两大类：

- GatewayFilter：应用到单个路由或者一个分组的路由上。
- GlobalFilter：应用到所有的路由上。



# 应用到单个路由上的



![img](../../../../\study-notes\sources\springcloud\webp)

AddRequestParameter 过滤器使用：
 `添加请求参数的过滤器`

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: gongjie
          uri: lb://APPUSER
          filters:
            - AddRequestParameter=name,gong
          predicates:
            - Cookie=CookieName, .*121.*
```





# 应用到全局的(自定义编写)

```java

@Component
public class LogGateWay implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        System.out.println("*******进如全局过滤器：当前时间：" + ZonedDateTime.now());

        String name = exchange.getRequest().getRemoteAddress().getHostName();

        //在这点就可以做一些过滤
        if(name == null){
            //如果条件不成立，这点就可以返回
            //设置返回状态
            exchange.getResponse().setRawStatusCode(HttpStatus.SC_BAD_GATEWAY);
            return exchange.getResponse().setComplete();
        }

        System.out.println("********链路继续执行");
        //条件成立的话  就让链路继续执行
        return chain.filter(exchange);
    }

    /**
     * 这里返回顺序，顺序越小，越在前面  范围就是Integer 的最大值和最小值
     * @return
     */
    @Override
    public int getOrder() {
        return 0;
    }
}

```

需要实现 GlobalFilter, Ordered两个接口