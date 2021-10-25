# 序列化

[`org.springframework.data.redis.serializer.RedisSerializer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/serializer/RedisSerializer.java) 接口，Redis 序列化接口，用于 Redis KEY 和 VALUE 的序列化。

```java
// RedisSerializer.java
public interface RedisSerializer<T> {

	@Nullable
	byte[] serialize(@Nullable T t) throws SerializationException;

	@Nullable
	T deserialize(@Nullable byte[] bytes) throws SerializationException;

}
```

- 定义了对象 `<T>` 和二进制数组的转换。
- 我们在 `redis-cli` 终端，看到的不都是字符串么，怎么这里是序列化成二进制数组呢？实际上，Redis Client 传递给 Redis Server 是传递的 KEY 和 VALUE 都是二进制值数组。好奇的胖友，可以打开 Jedis [`Connection#sendCommand(final Command cmd, final byte[\]... args)`](https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/Connection.java#L123) 方法，传入的参数就是二进制数组，而 `cmd` 命令也会被序列化成二进制数组。

RedisSerializer 的实现类，如下图：![Spring Data Redis 调用](../../../sources\img\02.png)

主要分成四类：

- JDK 序列化方式
- String 序列化方式
- JSON 序列化方式
- XML 序列化方式

### JDK 序列化方式

[`org.springframework.data.redis.serializer.JdkSerializationRedisSerializer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/serializer/JdkSerializationRedisSerializer.java) ，默认情况下，RedisTemplate 使用该数据列化方式。具体的，可以看看 [`RedisTemplate#afterPropertiesSet()`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/RedisTemplate.java) 方法，在 RedisTemplate 未设置序列化的情况下，使用 JdkSerializationRedisSerializer 作为序列化实现。在 Spring Boot 自动化配置 RedisTemplate Bean 对象时，就未设置。

绝大多数情况下，可能 99.9999% ，我们不会使用 JdkSerializationRedisSerializer 进行序列化。

```java
// Test01.java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01 {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void testStringSetKey02() {
        redisTemplate.opsForValue().set("aa", "bbb");
    }
}
```

我们先来执行下 `#testStringSetKey02()` 方法这个测试方法。注意，此处我们使用的是 RedisTemplate 而不是 StringRedisTemplate 。执行完成后，我们在控制台查询，看看是否真的执行成功了。

```shell
# 在 `redis-cli` 终端中

127.0.0.1:6379> scan 0
1) "0"
2) 1) "\xac\xed\x00\x05t\x00\x05aa"

127.0.0.1:6379> get "\xac\xed\x00\x05t\x00\x05aa"
"\xac\xed\x00\x05t\x00\x05bb"
```

- 通过 Redis [SCAN](http://redis.cn/commands/scan.html) 命令，我们扫描出了一个奇怪的 `"aa"` KEY ，前面带着奇怪的 16 进制字符。而后，我们使用这个奇怪的 KEY 去获取对应的 VALUE ，结果前面也是一串奇怪的 16 进制字符。

  > 具体为什么是这样一串奇怪的 16 进制，胖友可以看看 [`ObjectOutputStream#writeString(String str, boolean unshared)`](https://github.com/JetBrains/jdk8u_jdk/blob/master/src/share/classes/java/io/ObjectOutputStream.java#L1301-L1311) 的代码，实际就是标志位 + 字符串长度 + 字符串内容。

对于 KEY 被序列化成这样，我们线上通过 KEY 去查询对应的 VALUE 势必会非常不方便，所以 KEY 肯定是不能被这样序列化的。

对于 VALUE 被序列化成这样，除了阅读可能困难一点，不支持跨语言外，实际上也没啥问题。不过，实际线上场景，还是使用 JSON 序列化居多。



### ## String 序列化方式

① [`org.springframework.data.redis.serializer.StringRedisSerializer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/serializer/StringRedisSerializer.java) ，字符串和二进制数组的**直接**转换。代码如下：

```java
// StringRedisSerializer.java
private final Charset charset;

@Override
public String deserialize(@Nullable byte[] bytes) {
	return (bytes == null ? null : new String(bytes, charset));
}

@Override
public byte[] serialize(@Nullable String string) {
	return (string == null ? null : string.getBytes(charset));
}
```

- 是不是很直接简单。

**绝大多数情况下，我们 KEY 和 VALUE 都会使用这种序列化方案**。而 VALUE 的序列化和反序列化，自己在逻辑调用 JSON 方法去序列化。

② [`org.springframework.data.redis.serializer.GenericToStringSerializer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/serializer/GenericToStringSerializer.java) ，使用 Spring [ConversionService](https://codeday.me/bug/20180111/117294.html) 实现 `<T>` 对象和 String 的转换，从而 String 和二进制数组的转换。

例如说，序列化的过程，首先 `<T>` 对象通过 ConversionService 转换成 String ，然后 String 再序列化成二进制数组。反序列化的过程，胖友自己结合源码思考下  。

当然，GenericToStringSerializer 貌似基本不会去使用，所以不用去了解也问题不大。

### 3.1.3 JSON 序列化方式

① [`org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/serializer/GenericJackson2JsonRedisSerializer.java) ，使用 Jackson 实现 JSON 的序列化方式，并且从 Generic 单词可以看出，是支持所有类。怎么体现呢？参见构造方法的代码：

```java
// GenericJackson2JsonRedisSerializer.java

public GenericJackson2JsonRedisSerializer(@Nullable String classPropertyTypeName) {

	this(new ObjectMapper());

	// simply setting {@code mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS)} does not help here since we need
	// the type hint embedded for deserialization using the default typing feature.
	mapper.registerModule(new SimpleModule().addSerializer(new NullValueSerializer(classPropertyTypeName)));

	// <1>
	if (StringUtils.hasText(classPropertyTypeName)) {
		mapper.enableDefaultTypingAsProperty(DefaultTyping.NON_FINAL, classPropertyTypeName);
	// <2>
	} else {
		mapper.enableDefaultTyping(DefaultTyping.NON_FINAL, As.PROPERTY);
	}
}
```

- `<1>` 处，如果传入了 `classPropertyTypeName` 属性，就是使用使用传入对象的 `classPropertyTypeName` 属性对应的值，作为默认类型（Default Typing）。
- `<2>` 处，如果未传入 `classPropertyTypeName` 属性，则使用传入对象的类全名，作为默认类型（Default Typing）。

那么，胖友可能会问题，什么是**默认类型（Default Typing）**呢？我们来思考下，在将一个对象序列化成一个字符串，怎么保证字符串反序列化成对象的类型呢？Jackson 通过 Default Typing ，会在字符串多冗余一个类型，这样反序列化就知道具体的类型了。来举个例子，使用我们等会示例会用到的 [UserCacheObject](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/main/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/cacheobject/UserCacheObject.java) 类。

- 标准序列化的结果，如下：

  ```json
  {
      "id": 1,
      "name": "源码",
      "gender": 1
  }
  ```

- 使用 Jackson Default Typing 机制序列化的结果，如下：

  ```json
  {
      "@class": "cn..springdatarediswithjedis.cacheobject.UserCacheObject",
      "id": 1,
      "name": "源码",
      "gender": 1
  }
  ```

  - 看 `@class` 属性，反序列化的对象的类型不就有了么？

下面我们来看一个 GenericJackson2JsonRedisSerializer 的示例。小节，来看看如何配置 GenericJackson2JsonRedisSerializer 作为 VALUE 的序列化方式。然后，马上调回到此处。

示例代码如下：

```java
// Test01.java

@Autowired
private RedisTemplate redisTemplate;

@Test
public void testStringSetKeyUserCache() {
    UserCacheObject object = new UserCacheObject()
            .setId(1)
            .setName("源码")
            .setGender(1); // 男
    String key = String.format("user:%d", object.getId());
    redisTemplate.opsForValue().set(key, object);
}

@Test
public void testStringGetKeyUserCache() {
    String key = String.format("user:%d", 1);
    Object value = redisTemplate.opsForValue().get(key);
    System.out.println(value);
}
```

胖友分别执行 `#testStringSetKeyUserCache()` 和 `#testStringGetKeyUserCache()` 方法，然后对着 Redis 的结果看看，比较简单，就不多哔哔了。

我们在回过头来看看 `@class` 属性，它看似完美解决了反序列化后的对象类型，但是带来 JSON 字符串占用变大，所以实际项目中，我们也并不会采用 Jackson2JsonRedisSerializer 类。

② [`org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/serializer/GenericJackson2JsonRedisSerializer.java) ，使用 Jackson 实现 JSON 的序列化方式，并且显示指定 `<T>` 类型。代码如下：

```java
// Jackson2JsonRedisSerializer.java
public class Jackson2JsonRedisSerializer<T> implements RedisSerializer<T> {
    // ... 省略不重要的代码

    public static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;

    /**
     * 指定类型，和 <T> 要一致。
     */
    private final JavaType javaType;

    private ObjectMapper objectMapper = new ObjectMapper();

}
```

因为 Jackson2JsonRedisSerializer 序列化类里已经声明了类型，所以序列化的 JSON 字符串，无需在存储一个 `@class` 属性，用于存储类型。

但是，我们抠脚一想，如果使用 Jackson2JsonRedisSerializer 作为序列化实现类，那么如果我们类型比较多，岂不是每个类型都要定义一个 RedisTemplate Bean 了？！所以实际场景下，我们也并不会使用 Jackson2JsonRedisSerializer 类。😈

③ [`com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer`](https://github.com/alibaba/fastjson/blob/master/src/main/java/com/alibaba/fastjson/support/spring/GenericFastJsonRedisSerializer.java) ，使用 FastJSON 实现 JSON 的序列化方式，和 GenericJackson2JsonRedisSerializer 一致，就不重复赘述。

> 注意，GenericFastJsonRedisSerializer 不是 Spring Data Redis 内置实现，而是由于 FastJSON 自己实现。

④ [`com.alibaba.fastjson.support.spring.FastJsonRedisSerializer`](https://github.com/alibaba/fastjson/blob/master/src/main/java/com/alibaba/fastjson/support/spring/FastJsonRedisSerializer.java) ，使用 FastJSON 实现 JSON 的序列化方式，和 Jackson2JsonRedisSerializer 一致，就不重复赘述。

> 注意，GenericFastJsonRedisSerializer 不是 Spring Data Redis 内置实现，而是由于 FastJSON 自己实现。

###  XML 序列化方式

[`org.springframework.data.redis.serializer.OxmSerializer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/serializer/OxmSerializer.java) ，使用 Spring [OXM](https://www.jianshu.com/p/1b7c8dc26001) 实现将对象和 String 的转换，从而 String 和二进制数组的转换。

因为 XML 序列化方式，暂时没有这么干过，我自己也没有，所以就直接忽略它吧。😝

## 3.2 配置序列化方式

创建 RedisConfiguration 配置类，代码如下：

```java
@Configuration
public class RedisConfiguration {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 创建 RedisTemplate 对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置 RedisConnection 工厂。😈 它就是实现多种 Java Redis 客户端接入的秘密工厂。感兴趣的胖友，可以自己去撸下。
        template.setConnectionFactory(factory);

        // 使用 String 序列化方式，序列化 KEY 。
        template.setKeySerializer(RedisSerializer.string());

        // 使用 JSON 序列化方式（库是 Jackson ），序列化 VALUE 。
        template.setValueSerializer(RedisSerializer.json());
        return template;
    }

}
```

- `RedisSerializer#string()` 静态方法，返回的就是使用 UTF-8 编码的 StringRedisSerializer 对象。代码如下：

  ```java
  // RedisSerializer.java
  static RedisSerializer<String> string() {
  	return StringRedisSerializer.UTF_8;
  }
  
  // StringRedisSerializer.java
  public static final StringRedisSerializer ISO_8859_1 = new StringRedisSerializer(StandardCharsets.ISO_8859_1);
  ```

- `RedisSerializer#json()` 静态方法，返回 GenericJackson2JsonRedisSerializer 对象。代码如下：

  ```java
  // RedisSerializer.java
  
  static RedisSerializer<Object> json() {
  	return new GenericJackson2JsonRedisSerializer();
  }
  ```

## 自定义 RedisSerializer 实现类

我们直接以 GenericFastJsonRedisSerializer 举例子，直接莽源码。代码如下：

```java
// GenericFastJsonRedisSerializer.java

public class GenericFastJsonRedisSerializer implements RedisSerializer<Object> {
    private final static ParserConfig defaultRedisConfig = new ParserConfig();
    static { defaultRedisConfig.setAutoTypeSupport(true);}

    public byte[] serialize(Object object) throws SerializationException {
        // 空，直接返回空数组
        if (object == null) {
            return new byte[0];
        }
        try {
            // 使用 JSON 进行序列化成二进制数组，同时通过 SerializerFeature.WriteClassName 参数，声明写入类全名。
            return JSON.toJSONBytes(object, SerializerFeature.WriteClassName);
        } catch (Exception ex) {
            throw new SerializationException("Could not serialize: " + ex.getMessage(), ex);
        }
    }

    public Object deserialize(byte[] bytes) throws SerializationException {
        // 如果为空，则返回空对象
        if (bytes == null || bytes.length == 0) {
            return null;
        }
        try {
            // 使用 JSON 解析成对象。
            return JSON.parseObject(new String(bytes, IOUtils.UTF8), Object.class, defaultRedisConfig);
        } catch (Exception ex) {
            throw new SerializationException("Could not deserialize: " + ex.getMessage(), ex);
        }
    }
}
```

完成自定义 RedisSerializer 配置类后，将 VALUE 序列化的修改成我们的，哈哈哈。

