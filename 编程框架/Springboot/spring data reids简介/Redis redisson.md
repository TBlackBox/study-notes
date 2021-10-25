# 尝试 Redisson

可能胖友不是很了解 Redisson 这个库，胖友可以跳转 [Redis 客户端 Redisson](https://www.oschina.net/p/redisson) ，看看对它的介绍。简单来说，这是 **Java 最强的 Redis 客户端**！除了提供了 Redis 客户端的常见操作之外，还提供了 Redis 分布式锁、BloomFilter 布隆过滤器等强大的功能。

> 在 [redisson-examples](https://github.com/redisson/redisson-examples) 中，Redisson 官方提供了大量的示例。

## 6.1 快速入门

> 示例代码对应仓库：[spring-data-redis-with-redisson](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-redisson) 。

### 6.1.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-redisson/pom.xml) 中，引入相关依赖。

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.3.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>

    <!-- 实现对 Redisson 的自动化配置 --> <!-- X -->
    <dependency>
        <groupId>org.redisson</groupId>
        <artifactId>redisson-spring-boot-starter</artifactId>
        <version>3.11.3</version>
    </dependency>

    <!-- 方便等会写单元测试 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- 等会示例会使用 fastjson 作为 JSON 序列化的工具 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.61</version>
    </dependency>

    <!-- Spring Data Redis 默认使用 Jackson 作为 JSON 序列化的工具 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>

    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>

</dependencies>
```

和 [「2.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/Redis/?yudao#) 的差异点是，我们需要引入 `redisson-spring-boot-starter` 依赖，实现 Redisson 的自动化配置。

### 6.1.2 配置文件

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-redisson/src/main/resources/application.yml) 中，添加 Redis 配置，如下：

```yaml
spring:
  # 对应 RedisProperties 类
  redis:
    host: 127.0.0.1
    port: 6379
#    password: # Redis 服务器密码，默认为空。生产中，一定要设置 Redis 密码！
    database: 0 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
    # 对应 RedissonProperties 类
    redisson:
      config: classpath:redisson.yml # 具体的每个配置项，见 org.redisson.config.Config 类。
```

和 [「2.2 配置文件」](https://www.iocoder.cn/Spring-Boot/Redis/?yudao#) 的差异点是：

**1）去掉 Jedis 相关的配置项**

**2）增加 `redisson.config` 配置**

在我们使用 Spring Boot 整合 Redisson 时候，通过该配置项，引入一个外部的 Redisson 相关的配置文件。例如说，示例中，我们引入了 `classpath:redisson.yaml` 配置文件。它可以使用 JSON 或 YAML 格式，进行配置。

而引入的 `redisson.config` 对应的配置文件，对应的类是 [`org.redisson.config.Config`](https://github.com/redisson/redisson/blob/master/redisson/src/main/java/org/redisson/config/Config.java) 类。因为示例中，我们使用的比较简单，所以就没有做任何 Redisson 相关的自定义配置。如下是 Redisson 的每个配置项的解释：

> FROM [《Spring Boot 2.x 整合 lettuce redis 和 redisson》](https://blog.csdn.net/zl_momomo/article/details/82788294) 文章。

```properties
clusterServersConfig:
  # 连接空闲超时 如果当前连接池里的连接数量超过了最小空闲连接数，而同时有连接空闲时间超过了该数值，那么这些连接将会自动被关闭，并从连接池里去掉。时间单位是毫秒。
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  # 连接超时
  connectTimeout: 10000
  # 命令等待超时
  timeout: 3000
  # 命令失败重试次数
  retryAttempts: 3
  # 命令重试发送时间间隔
  retryInterval: 1500
  # 重新连接时间间隔
  reconnectionTimeout: 3000
  # failedAttempts
  failedAttempts: 3
  # 密码
  password: null
  # 单个连接最大订阅数量
  subscriptionsPerConnection: 5
  # 客户端名称
  clientName: null
  #负载均衡算法类的选择  默认轮询调度算法RoundRobinLoadBalancer
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  slaveSubscriptionConnectionMinimumIdleSize: 1
  slaveSubscriptionConnectionPoolSize: 50
  # 从节点最小空闲连接数
  slaveConnectionMinimumIdleSize: 32
  # 从节点连接池大小
  slaveConnectionPoolSize: 64
  # 主节点最小空闲连接数
  masterConnectionMinimumIdleSize: 32
  # 主节点连接池大小
  masterConnectionPoolSize: 64
  # 只在从服务节点里读取
  readMode: "SLAVE"
  # 主节点信息
  nodeAddresses:
  - "redis://192.168.56.128:7000"
  - "redis://192.168.56.128:7001"
  - "redis://192.168.56.128:7002"
  #集群扫描间隔时间 单位毫秒
  scanInterval: 1000
threads: 0
nettyThreads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
```

如果 `redisson.config` 对应的配置文件，如果没有配置任何内容，需要在 `application.yml` 里注释掉 `redisson.config` 。像这样：

```properties
spring:
  # 对应 RedisProperties 类
  redis:
    host: 127.0.0.1
    port: 6379
#    password: # Redis 服务器密码，默认为空。生产中，一定要设置 Redis 密码！
    database: 0 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
    # 对应 RedissonProperties 类
#    redisson:
#      config: classpath:redisson.yml # 具体的每个配置项，见 org.redisson.config.Config 类。
```

### 6.1.3 简单测试

创建 [Test01](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-redisson/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/Test01.java) 测试类，我们来测试一下简单的 SET 指令。代码如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01 {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void testStringSetKey() {
        stringRedisTemplate.opsForValue().set("yunai", "shuai");
    }
}
```

我们先来执行下 `#testStringSetKey()` 方法这个测试方法。执行完成后，我们在控制台查询，看看是否真的执行成功了。

```
$ redis-cli get yunai
"shuai"
```

- 请大声的告诉我，Redis 是怎么夸奖 `"yunai"` 的，哈哈哈哈。

### 6.1.4 闲聊两句

因为有 Spring Data Redis 的存在，我们其实已经能感受到，即使我们将 Jedis 替换成了 Redisson ，依然调用的是相同的 Spring Data Redis 提供的 API ，而无需感知到 Redisson 或是 Jedis 的存在。如果哪一天，Spring Boot 2.X 版本默认推荐的 Lettuce 真的成熟了，那么我们也可以无感知的进行替换。