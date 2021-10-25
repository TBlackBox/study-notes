## Transaction

> 基情提示：实际项目实战中，Redis Transaction 事务基本不用，至少问了一些胖友，包括自己，都没有再用。所以呢，本小节可以选择性看看。或者，就不看，哈哈哈哈。

在看 Redis Transaction 事务之前，我们先回想下 Spring 是如何管理**数据库 Transaction** 的。在应用程序中处理一个请求时，如果我们的方法开启Trasaction 功能，Spring 会把数据库的 Connection 连接和当前线程进行绑定，从而实现 Connection 打开一个 Transaction 后，所有当前线程的数据库操作都在该 Connection 上执行，达到所有操作在这个 Transaction 中，最终提交或回滚。

在 Spring Data Redis 中，实现 Redis Transaction 也是这个思路。通过 SessionCallback 操作 Redis 时，会从当前线程获得 Redis Connection ，如果获取不到，则会去“创建”一个 Redis Connection 并绑定到当前线程中。这样，我们在该 Redis Connection 开启 Redis Transaction 后，在该线程的所有操作，都可以在这个 Transaction 中，最后交由 Spring 事务管理器统一提供或回滚 Transaction 。

如果想要使用 Redis Transaction 功能，需要创建 RedisTemplate Bean 时，设置其 [`enableTransactionSupport`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/RedisTemplate.java#L91) 属性为 `true` ，默认为 `false` 不开启。示例如下：

```java
@Configuration
public class RedisConfiguration {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 创建 RedisTemplate 对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();

        // 【重要】设置开启事务支持
        template.setEnableTransactionSupport(true);

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

### 5.2.1 源码解析

概念和原理层面的东西，一旦复杂，就会特别抽象，那么还是老规矩，让我们一起撸下源码，让原理具象化。很多时候，这就是为什么我们要去撸源码的意义。

我们先来看看，配置下 `enableTransactionSupport` 属性，Redis 在执行命令，是如何获得 Connection 连接的。代码如下：

```java
// RedisTemplate.java

public <T> T execute(RedisCallback<T> action, boolean exposeConnection, boolean pipeline) {

	Assert.isTrue(initialized, "template not initialized; call afterPropertiesSet() before using it");
	Assert.notNull(action, "Callback object must not be null");

	RedisConnectionFactory factory = getRequiredConnectionFactory();
	RedisConnection conn = null;
	try {
		// <1.1>
		if (enableTransactionSupport) {
			// only bind resources in case of potential transaction synchronization
			conn = RedisConnectionUtils.bindConnection(factory, enableTransactionSupport);
		} else {
			// <1.2>
			conn = RedisConnectionUtils.getConnection(factory);
		}

		// ... 省略中间，执行 Redis 命令的代码。
	} finally {
		// <2>
		RedisConnectionUtils.releaseConnection(conn, factory);
	}
}
```

- 考虑到尽量让内容简单一些，我们不会对每一行代码做特别的深究，主要是保证胖友对 Spring Data Redis 对 Transaction 的封装，有个总体了解。

- `<1.2>` 处，当我们**未**开启 `enableTransactionSupport` 事务时，调用 [`RedisConnectionUtils#getConnection(factory)`](https://github.com/spring-projects/spring-data-redis/blob/64b89137648f6c0e0c810c624e481bcfc0732f4e/src/main/java/org/springframework/data/redis/core/RedisConnectionUtils.java#L81) 方法，获得 Redis Connection 。如果获取不到，则进行创建。

- <1.1>处，当我们有开启
  ```
  enableTransactionSupport
  ```
  事务时，调用
  `RedisConnectionUtils#bindConnection(RedisConnectionFactory factory, boolean enableTranactionSupport)`

  方法，在

  ```
  RedisConnectionUtils#getConnection(factory)
  ```

  的基础上，如果是创建的 Redis Connection ，会绑定到当前啊线程中。因为 Transaction 是需要在 Connection 打开，然后后续的 Redis 的操作，都需要在其上。并且，还有一个非常重要的操作，打开 Redis Transaction ，会在该方法中，通过调用 [`RedisConnectionUtils#potentiallyRegisterTransactionSynchronisation(RedisConnectionHolder connHolder,
  
  ```
  final RedisConnectionFactory factory)`](https://github.com/spring-projects/spring-data-redis/blob/64b89137648f6c0e0c810c624e481bcfc0732f4e/src/main/java/org/springframework/data/redis/core/RedisConnectionUtils.java#L163) 。
  ```

- `<2>` 处，调用 [`RedisConnectionUtils#releaseConnection(RedisConnection conn, RedisConnectionFactory factory)`](https://github.com/spring-projects/spring-data-redis/blob/64b89137648f6c0e0c810c624e481bcfc0732f4e/src/main/java/org/springframework/data/redis/core/RedisConnectionUtils.java#L206) 方法，释放 Redis Connection 。当然，这是有一个前提，整个 Transaction 已经完成。如果未完成，实际 Redis Connection 不会释放。

那么，此时会有胖友有疑问，Redis Transaction 的提交和回滚在哪呢？答案在 RedisConnectionUtils 的内部类 RedisTransactionSynchronizer 中。代码如下：

```java
// RedisConnectionUtils.java

private static class RedisTransactionSynchronizer extends TransactionSynchronizationAdapter {

	private final RedisConnectionHolder connHolder;
	private final RedisConnection connection;
	private final RedisConnectionFactory factory;

	@Override
	public void afterCompletion(int status) {

		try {
			switch (status) {
				// 提交
				case TransactionSynchronization.STATUS_COMMITTED:
					connection.exec();
					break;
				// 回滚
				case TransactionSynchronization.STATUS_ROLLED_BACK:
				case TransactionSynchronization.STATUS_UNKNOWN:
				default:
					connection.discard();
			}
		} finally {
			connHolder.setTransactionSyncronisationActive(false);
			connection.close();
			TransactionSynchronizationManager.unbindResource(factory);
		}
	}
}
```

- 根据事务结果的状态，进行 Redis Transaction 提交或回滚。😈 如果想进一步的深入，胖友就需要去了解 Spring Transaction 的源码。

###  具体示例

```java
// TransactionTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
public class TransactionTest {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
//    @Transactional
    public void test01() {
        // 这里是偷懒，没在 RedisConfiguration 配置类中，设置 stringRedisTemplate 开启事务。
        stringRedisTemplate.setEnableTransactionSupport(true);

        // 执行想要的操作
        stringRedisTemplate.opsForValue().set("aa:1", "aa");
        stringRedisTemplate.opsForValue().set("sss:1", "aaaa");
    }
}
```

目前这仅仅是一个示例。因为 Redis Transaction **实际**创建事务的前提，是当前已经存在 Spring Transaction 。**具体可以看看[传送门](https://github.com/spring-projects/spring-data-redis/blob/64b89137648f6c0e0c810c624e481bcfc0732f4e/src/main/java/org/springframework/data/redis/core/RedisConnectionUtils.java#L159)处的判断的代码**。😈 略感神奇，不晓得为什么是这样的设定。

### 5.2.3 补充资料

如果觉得还是无法理解的胖友，可以在看看如下几篇文章：

- [《Spring Data Redis(Redis Transactions)》](https://blog.csdn.net/zlfprogram/article/details/75196156)
- [《Redis 之坑：spring-data-redis 中的 Redis 事务》](https://blog.csdn.net/qq_32331073/article/details/79899762)
- [《Spring Data Redis 事务专题》](https://www.cnblogs.com/softidea/p/5720938.html)

### 5.2.4 闲话两句

实际场景下，如果胖友有 Redis 事务的诉求，建议把事务的、和非事务的 RedisTemplate 拆成两个连接池，相互独立。主要原因有两个：

- 1）Spring Data Redis 的事务设计，是将其融入到 Spring 整个 Transaction 当中。一般来说，Spring Transaction 中，肯定会存在数据库的 Transaction 。考虑到数据库操作相比 Redis 来说，肯定是慢得多，那么就会导致 Redis 的 Connection 一直被当前 Transaction 占用着。
- 2）[How can i eliminate getting junk value through redis get command?](https://stackoverflow.com/questions/34992256/how-can-i-eliminate-getting-junk-value-through-redis-get-command)