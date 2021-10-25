# 简介

如果没有了解过 Redis 的 Pipeline 机制，可以看看 [《Redis 文档 —— Pipeline》](http://redis.cn/topics/pipelining.html) 文章，批量操作，提升性能必备神器。

在 RedisTemplate 类中，提供了 2 组四个方法，用于执行 Redis Pipeline 操作。代码如下：

```java
// <1> 基于 Session 执行 Pipeline
@Override
public List<Object> executePipelined(SessionCallback<?> session) {
	return executePipelined(session, valueSerializer);
}
@Override
public List<Object> executePipelined(SessionCallback<?> session, @Nullable RedisSerializer<?> resultSerializer) {
    // ... 省略代码
}

// <2> 直接执行 Pipeline
@Override
public List<Object> executePipelined(RedisCallback<?> action) {
	return executePipelined(action, valueSerializer);
}
@Override
public List<Object> executePipelined(RedisCallback<?> action, @Nullable RedisSerializer<?> resultSerializer) {
    // ... 省略代码
}
```

- 两组方法的差异，在于是否是 Session 中执行。那么 Session 是什么呢？卖个关子。本小节，我们只讲 Pipeline + RedisCallback 的组合的方法。
- 每组方法里，差别在于是否传入 RedisSerializer 参数。如果不传，则使用 RedisTemplate 自己的序列化相关的属性。

### 5.1.1 源码解读

在看具体的 `#executePipelined(RedisCallback<?> action, ...)` 方法的示例之前，我们先来看一波源码，这样我们才能更好的理解具体的使用方法。代码如下：

```java
// RedisTemplate.java
@Override
public List<Object> executePipelined(RedisCallback<?> action, @Nullable RedisSerializer<?> resultSerializer) {
	// <1> 执行 Redis 方法
	return execute((RedisCallback<List<Object>>) connection -> {
		// <2> 打开 pipeline
		connection.openPipeline();
		boolean pipelinedClosed = false; // 标记 pipeline 是否关闭
		try {
			// <3> 执行
			Object result = action.doInRedis(connection);
			// <4> 不要返回结果
			if (result != null) {
				throw new InvalidDataAccessApiUsageException(
						"Callback cannot return a non-null value as it gets overwritten by the pipeline");
			}
			// <5> 提交 pipeline 执行
			List<Object> closePipeline = connection.closePipeline();
			pipelinedClosed = true;
			// <6> 反序列化结果，并返回
			return deserializeMixedResults(closePipeline, resultSerializer, hashKeySerializer, hashValueSerializer);
		} finally {
			if (!pipelinedClosed) {
				connection.closePipeline();
			}
		}
	});
}
```

- `<1>` 处，调用 `#execute(RedisCallback<T> action)` 方法，执行 Redis 方法。注意，此处传入的 `action` 参数，不是我们传入的 RedisCallback 参数。我们的会在该 `action` 中被执行。
- `<2>` 处，调用 `RedisConnection#openPipeline()` 方法，**自动**打开 Pipeline 模式。这样，我们就不需要手动去打开了。
- `<3>` 处，调用我们传入的实现的 `RedisCallback#doInRedis(RedisConnection connection)` 方法，执行在 Pipeline 中，想要执行的 Redis 操作。
- `<4>` 处，不要返回结果。因为 RedisCallback 是统一定义的接口，所以可以返回一个结果。但是在 Pipeline 中，未提交执行时，显然是没有结果，返回也没有意思。简单来说，就是我们在实现 `RedisCallback#doInRedis(RedisConnection connection)` 方法时，返回 `null` 即可。
- `<5>` 处，调用 `RedisConnection#closePipeline()` 方法，**自动**提交 Pipeline 执行，并返回执行结果。
- `<6>` 处，反序列化结果，并返回 Pipeline 结果。

至此，Spring Data Redis 对 Pipeline 的封装，我们已经做了一个简单的了解，实际就是经典的[“模板方法”](http://www.iocoder.cn/DesignPattern/xiaomingge/Template-Method/?vip)设计模式化的应用。下面，在让我们来看看 [`org.springframework.data.redis.core.RedisCallback`](https://github.com/spring-projects/spring-data-redis/blob/2eb7067e8c7e859168a281145cc46ccddb42049f/src/main/java/org/springframework/data/redis/core/RedisCallback.java) 接口，Redis 回调接口。代码如下：

```java
// RedisCallback.java
public interface RedisCallback<T> {

	/**
	 * Gets called by {@link RedisTemplate} with an active Redis connection. Does not need to care about activating or
	 * closing the connection or handling exceptions.
	 *
	 * @param connection active Redis connection
	 * @return a result object or {@code null} if none
	 * @throws DataAccessException
	 */
	@Nullable
	T doInRedis(RedisConnection connection) throws DataAccessException;
}
```

- 虽然接口名是以 Callback 结尾，但是通过 `#doInRedis(RedisConnection connection)` 方法可以很容易知道，实际可以理解是 Redis Action ，想要执行的 Redis 操作。

- 有一点要注意，传入的 `connection` 参数是 RedisConnection 对象，它提供的 `'low level'` 更底层的 Redis API 操作。例如说：

  ```java
  // RedisStringCommands.java
  // RedisConnection 实现 RedisStringCommands 接口
  
  byte[] get(byte[] key);
  
  Boolean set(byte[] key, byte[] value);
  ```

  - 传入和返回的是二进制数组，实际就是 RedisTemplate 已经序列化的入参和会被反序列化的出参。

### 5.1.2 具体示例

创建 [PipelineTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/PipelineTest.java) 单元测试类，编写代码如下：

```java
// PipelineTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
public class PipelineTest {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() {
        List<Object> results = stringRedisTemplate.executePipelined(new RedisCallback<Object>() {

            @Override
            public Object doInRedis(RedisConnection connection) throws DataAccessException {
                // set 写入
                for (int i = 0; i < 3; i++) {
                    connection.set(String.format("aa:%d", i).getBytes(), "aa".getBytes());
                }

                // get
                for (int i = 0; i < 3; i++) {
                    connection.get(String.format("aa:%d", i).getBytes());
                }

                // 返回 null 即可
                return null;
            }
        });

        // 打印结果
        System.out.println(results);
    }
}
```

执行 `#test01()` 方法，结果如下：

```
[true, true, true, aa, aa, aa]
```

- 因为我们使用 StringRedisTemplate 自己的序列化相关属性，所以 Redis GET 命令返回的二进制，被反序列化成了字符串。