## Session

首先，我们需要澄清下，Session 不是 Redis 的功能，而是 Spring Data Redis 封装的一个功能。一次 Session ，代表通过同一个 Redis Connection 执行一系列的 Redis 操作。

我们可以发现，如果我们在一个 Redis Transaction 中的时候，所有 Redis 操作都使用通过同一个 Redis Connection ，因为我们会将获得到的 Connection 绑定到当前线程中。

但是，如果我们不在一个 Redis Transaction 中的时候，我们每一次使用 Redis Operations 执行 Redis 操作的时候，每一次都会获取一次 Redis Connection 的获取。实际项目中，我们必然会使用 Redis Connection 连接池，那么在获取的时候，会存在一定的竞争，会有资源上的消耗。那么，如果我们希望如果已知我们要执行一个系列的 Redis 操作，能不能使用同一个 Redis Connection ，避免重复获取它呢？答案是有，那就是 Session 。

当我们要执行在同一个 Session 里的操作时，我们通过实现 [`org.springframework.data.redis.core.SessionCallback`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/SessionCallback.java) 接口，其代码如下：

```java
// SessionCallback.java

public interface SessionCallback<T> {

	@Nullable
	<K, V> T execute(RedisOperations<K, V> operations) throws DataAccessException;
}
```

- 相比 RedisCallback 来说，总比是比较相似的。但是比较友好的是，它的入参 `operations` 是 [org.springframework.data.redis.core.RedisOperations](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/RedisOperations.java) 接口类型，而 RedisTemplate 的各种操作，实际就是在 RedisOperations 接口中定义，由 RedisTemplate 来实现。所以使用上也会更加便利。
- 实际上，我们在实现 RedisCallback 接口，也能实现在同一个 Connection 执行一系列的 Redis 操作，因为 RedisCallback 的入参本身就是一个 Redis Connection 。

### 5.3.1 源码解析

在生产中，Transaction 和 Pipeline 会经常一起时候用，从而提升性能。所以在 `RedisTemplate#executePipelined(SessionCallback<?> session, ...)` 方法中，提供了这种的功能。而在这个方法的实现上，本质和 `RedisTemplate#executePipelined(RedisCallback<?> action, ...)` 方法是基本一致的，差别在于[这一行](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/RedisTemplate.java#L289) ，替换成了调用 `#executeSession(SessionCallback<?> session)` 方法。所以，我们来直接来看被调用的这个方法的实现。代码如下：

```java
// RedisTemplate.java

@Override
public <T> T execute(SessionCallback<T> session) {

	Assert.isTrue(initialized, "template not initialized; call afterPropertiesSet() before using it");
	Assert.notNull(session, "Callback object must not be null");

	RedisConnectionFactory factory = getRequiredConnectionFactory();
	// bind connection
	// <1> 获得并绑定 Connection 。
	RedisConnectionUtils.bindConnection(factory, enableTransactionSupport);
	try {
	   // <2> 执行定义的一系列 Redis 操作
		return session.execute(this);
	} finally {
		// <3> 释放并解绑 Connection 。
		RedisConnectionUtils.unbindConnection(factory);
	}
}
```

- <1> 处，调用
  ```
  RedisConnectionUtils#bindConnection(RedisConnectionFactory factory, boolean enableTranactionSupport)
  ```方法，实际和我们开启
  ```
  enableTranactionSupport
  ```
  事务时候，获取 Connection 和处理的方式，是
  
  一模一样
  
  的。也就是说：
  
  - 如果当前线程已经有一个绑定的 Connection 则直接使用（例如说，当前正在 Redis Transaction 事务中）；
  - 如果当前线程未绑定一个 Connection ，则进行创建并绑定到当前线程。甚至，如果此时是配置开启 `enableTranactionSupport` 事务的，那么此处就会触发 Redis Transaction 的开启。

- `<2>` 处，调用 `SessionCallback#execute(RedisOperations<K, V> operations)` 方法，执行我们定义的一系列的 Redis 操作。看看此处传入的参数是 `this` ，是不是仿佛更加明白点什么了？

- `<3>` 处，调用 [`RedisConnectionUtils#unbindConnection(RedisConnectionFactory factory)`](https://github.com/spring-projects/spring-data-redis/blob/64b89137648f6c0e0c810c624e481bcfc0732f4e/src/main/java/org/springframework/data/redis/core/RedisConnectionUtils.java#L253) 方法，释放并解绑 Connection 。当前，前提是当前不存在激活的 Redis Transaction ，不然不就提早释放了嘛。

恩，现在胖友在回过头，好好在想一想 Pipeline、Transaction、Session 之间的关系，以及组合排列。之后，我们在使用上，会更加得心应手。

### 5.3.2 具体示例

> 示例代码对应测试类：[SessionTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/SessionTest.java) 。

创建 [SessionTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/SessionTest.java) 单元测试类，编写代码如下：

```java
// SessionTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
public class SessionTest {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() {
        String result = stringRedisTemplate.execute(new SessionCallback<String>() {

            @Override
            public String execute(RedisOperations operations) throws DataAccessException {
                for (int i = 0; i < 100; i++) {
                    operations.opsForValue().set(String.format("yunai:%d", i), "shuai02");
                }
                return (String) operations.opsForValue().get(String.format("yunai:%d", 0));
            }

        });

        System.out.println("result:" + result);
    }

}
```

执行 `#test01()` 方法，结果如下：

```
result:shuai02
```

- 卧槽，一直被 Redis 夸奖，已经超级不好意思了。