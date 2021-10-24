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
