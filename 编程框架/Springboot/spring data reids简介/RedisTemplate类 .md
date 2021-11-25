# 简介

提供 Redis 操作模板 API 。

## 核心代码

```java
//一些控制属性
private boolean enableTransactionSupport = false;
private boolean exposeConnection = false;
private boolean initialized = false;
private boolean enableDefaultSerializer = true;
private @Nullable RedisSerializer<?> defaultSerializer;
private @Nullable ClassLoader classLoader;

// A 序列化相关属性
@SuppressWarnings("rawtypes") private @Nullable RedisSerializer keySerializer = null;
@SuppressWarnings("rawtypes") private @Nullable RedisSerializer valueSerializer = null;
@SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashKeySerializer = null;
@SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashValueSerializer = null;
private RedisSerializer<String> stringSerializer = RedisSerializer.string();

//Lua 脚本执行器
private @Nullable ScriptExecutor<K> scriptExecutor;

//常见的数据结构操作
private final ValueOperations<K, V> valueOps = new DefaultValueOperations<>(this);
private final ListOperations<K, V> listOps = new DefaultListOperations<>(this);
private final SetOperations<K, V> setOps = new DefaultSetOperations<>(this);
private final StreamOperations<K, ?, ?> streamOps = new DefaultStreamOperations<>(this, new ObjectHashMapper());
private final ZSetOperations<K, V> zSetOps = new DefaultZSetOperations<>(this);
private final GeoOperations<K, V> geoOps = new DefaultGeoOperations<>(this);
private final HyperLogLogOperations<K, V> hllOps = new DefaultHyperLogLogOperations<>(this);
private final ClusterOperations<K, V> clusterOps = new DefaultClusterOperations<>(this);

```

- A 处，看到了四个序列化相关的属性，用于 KEY 和 VALUE 的序列化。
  - 例如说，我们在使用 POJO 对象存储到 Redis 中，一般情况下，会使用 JSON 方式序列化成字符串，存储到 Redis 中。。
  - 在springboot使用中，我们看到了 [`org.springframework.data.redis.core.StringRedisTemplate`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/StringRedisTemplate.java) 类，它继承 RedisTemplate 类，使用 [`org.springframework.data.redis.serializer.StringRedisSerializer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/serializer/StringRedisSerializer.java) 字符串序列化方式。直接点开 StringRedisSerializer 源码，看下它的构造方法。
- B处，Lua 脚本执行器，提供 [Redis scripting](http://redis.cn/commands.html#scripting) API 操作。
- C处，Redis 常见数据结构操作类。
  - [ValueOperations](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/ValueOperations.java) 类，提供 [Redis String](http://redis.cn/commands.html#string) API 操作。
  - [ListOperations](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/ListOperations.java) 类，提供 [Redis List](http://redis.cn/commands.html#list) API 操作。
  - [SetOperations](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/SetOperations.java) 类，提供 [Redis Set](http://redis.cn/commands.html#set) API 操作。
  - [ZSetOperations](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/ZSetOperations.java) 类，提供 [Redis ZSet(Sorted Set)](http://redis.cn/commands.html#sorted_set) API 操作。
  - [GeoOperations](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/GeoOperations.java) 类，提供 [Redis Geo](http://redis.cn/commands.html#geo) API 操作。
  - [HyperLogLogOperations](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/HyperLogLogOperations.java) 类，提供 [Redis HyperLogLog](http://redis.cn/commands.html#hyperloglog) API 操作。

那么 Pub/Sub、Transaction、Pipeline、Keys、Cluster、Connection 等相关的 API 操作呢？它在 RedisTemplate 自身提供，因为它们不属于具体每一种数据结构，所以没有封装在对应的 Operations 类中。



