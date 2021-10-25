# 简介
springboot提供了许多的缓存类型，redis缓存配置只需要引入starter,修改相应的配置文件即可。

# 配置
1. 添加starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. 修改配置文件

```java
spring:
  redis:
    host: 192.168.0.110
```

3. 使用方法

spingboot 默认有2个模板提供使用
+ RedisTemplate    可以存对象 ，默认的序列化方式是jdk的序列化方式
+ StringRedisTemplate 只能存字符串


使用时，只需要注入即可,下面只演示了对象的存储，字符串同理
```java
@Autowired
RedisTemplate redisTemplate;

@Autowired
StringRedisTemplate stringRedisTemplate;

@Test
public void contextLoads() {
    User user = new User();
    user.setId(1);
    user.setName("哈哈");
    redisTemplate.opsForValue().set("user",user);
    User user1 = (User) redisTemplate.opsForValue().get("user");
    System.out.println(user1.getName());
}
```

# 序列化json存储

1. 新建JsonRedisTemplate
```java
public class JsonRedisTemplate extends RedisTemplate {
}
```

2. 创建redis配置类
```java
@Configuration
public class RedisConfigtion {
    @Bean
    public JsonRedisTemplate cacheManager(RedisConnectionFactory redisConnectionFactory,
                                          ResourceLoader resourceLoader) {
        JsonRedisTemplate redisTemplate = new JsonRedisTemplate();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //设置键的序列化为字符串
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //设置值得序列化为JSON
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
        return redisTemplate;
    }
}
```

**注意：如果需要序列化其他类型，改变序列化参数即可**

3. 使用例子

```java
@Autowired
JsonRedisTemplate jsonRedisTemplate;

@Test
public void testRedis() throws JSONException {
    User user = new User();user.setId(1);
    user.setName("哈哈");
    jsonRedisTemplate.opsForValue().set("user",user);
    Object obj = jsonRedisTemplate.opsForValue().get("user");
    LinkedHashMap map = (LinkedHashMap) obj;
    JSONObject json = new JSONObject(map);
    System.out.println(json.getString("name"));
}
```

# 总结
这只是一个简单的配置用法，springboot的详细用法需要参照其他文档使用。

一篇文章

https://www.iocoder.cn/Spring-Boot/Redis/?yudao



# spring boot 实现对redis 的访问





# 概述

Redis、Redisson、Lettuce 等优秀的 Java Redis 工具库，为什么还要有 Spring Data Redis。看下面的图，都明白了。

![01](../../../sources\img\01.png)

- 对于下层，Spring Data Redis 提供了统一的操作模板（后文中，我们会看到是 RedisTemplate 类），封装了 Jedis、Lettuce 的 API 操作，访问 Redis 数据。所以，**实际上，Spring Data Redis 内置真正访问的实际是 Jedis、Lettuce 等 API 操作**。
- 对于上层，开发者学习如何使用 Spring Data Redis 即可，而无需关心 Jedis、Lettuce 的 API 操作。甚至，未来如果我们想将 Redis 访问从 Jedis 迁移成 Lettuce 来，无需做任何的变动。如果以后选择redis 客户端，换redis客户端依赖就可以了。
- 目前，Spring Data Redis 暂时只支持 Jedis、Lettuce 的内部封装，而 Redisson 是由 [redisson-spring-data](https://github.com/redisson/redisson/tree/master/redisson-spring-data) 来提供。



# spring boot中快速使用redis 

在 `spring-boot-starter-data-redis` 项目 2.X 中，默认使用 Lettuce 作为 Java Redis 工具库，猜测是因为 Jedis 中间有一段时间诈尸，基本不太更新。

考虑到自己项目中，使用 Jedis 为主，并且问了几个朋友，都是使用 Jedis ，并且有吐槽 Lettuce 坑多多，所以个人推荐的话，生产中还是使用 Jedis ，稳定第一。也因此，本节我们是 Spring Data Redis + Jedis 的组合。

## 引入依赖

```xml
<dependencies>

    <!-- 实现对 Spring Data Redis 的自动化配置 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <exclusions>
            <!-- 去掉对 Lettuce 的依赖，因为 Spring Boot 优先使用 Lettuce 作为 Redis 客户端 -->
            <exclusion>
                <groupId>io.lettuce</groupId>
                <artifactId>lettuce-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- 引入 Jedis 的依赖，这样 Spring Boot 实现对 Jedis 的自动化配置 -->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
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

</dependencies>
```

## 配置配置文件

```yaml
spring:
  # 对应 RedisProperties 类
  redis:
    host: 127.0.0.1
    port: 6379
    password: # Redis 服务器密码，默认为空。生产中，一定要设置 Redis 密码！
    database: 0 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
    # 对应 RedisProperties.Jedis 内部类
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数，默认为 8 。使用负数表示没有限制。
        max-idle: 8 # 默认连接数最小空闲的连接数，默认为 8 。使用负数表示没有限制。
        min-idle: 0 # 默认连接池最小空闲的连接数，默认为 0 。允许设置 0 和 正数。
        max-wait: -1 # 连接池最大阻塞等待时间，单位：毫秒。默认为 -1 ，表示不限制。
```

## 测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01 {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void testStringSetKey() {
        stringRedisTemplate.opsForValue().set("aa", "哈哈");
    }
}
```

通过 StringRedisTemplate 类，我们进行了一次 Redis SET 指令的执行。
