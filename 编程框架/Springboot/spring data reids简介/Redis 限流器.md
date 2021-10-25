## Redis 限流器

在开始本节之前，先推荐看一篇干货 [《你应该如何正确健壮后端服务？》](http://www.iocoder.cn/Fight/How-do-you-robust-back-end-services/?vip) 。

限流，无论在系统层面，还是在业务层面，使用都非常广泛。例如说：

- 【业务】为了避免恶意的灌水机或者用户，限制每分钟至允许回复 10 个帖子。
- 【系统】为了避免服务系统被大规模调用，超过极限，限制每个调用方只允许每秒调用 100 次。

限流算法，常用的分成四种：

> 每一种的概念，推荐看看 [《计数器、滑动窗口、漏桶、令牌算法比较和伪代码实现》](https://www.iphpt.com/detail/106) 文章。

- 计数器

  > 比较简单，每**固定单位**一个计数器即可实现。

- 滑动窗口

  > Redisson 提供的是基于**滑动窗口** RateLimiter 的实现。相比**计数器**的实现，它的起点不是固定的，而是以开始计数的那个时刻开始为一个窗口。
  >
  > 所以，我们可以把**计数器**理解成一个滑动窗口的特例，以**固定单位**为一个窗口。

- 令牌桶算法

  > [《Eureka 源码解析 —— 基于令牌桶算法的 RateLimiter》](http://www.iocoder.cn/Eureka/rate-limiter/?vip) ，单机并发场景下的 RateLimiter 实现。
  >
  > [《Spring-Cloud-Gateway 源码解析 —— 过滤器 (4.10) 之 RequestRateLimiterGatewayFilterFactory 请求限流》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-request-rate-limiter/?vip) ，基于 Redis 实现的令牌桶算法的 RateLimiter 实现。

- 漏桶算法

  > 漏桶算法，一直没搞明白和令牌桶算法的区别。现在的理解是：
  >
  > - 令牌桶算法，桶里装的是令牌。每次能拿取到令牌，就可以进行访问。并且，令牌会按照速率不断恢复放到令牌桶中直到桶满。
  > - 漏桶算法，桶里装的是请求。当桶满了，请求就进不来。例如说，Hystrix 使用线程池或者 Semaphore 信号量，只有在请求未满的时候，才可以进行执行。

上面哔哔了非常多的字，只看本文的话，就那一句话：“**Redisson 提供的是基于滑动窗口 RateLimiter 的实现。**”。

### 6.3.1 具体示例

> 示例代码对应测试类：[PubSubTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/RateLimiterTest.java) 。

创建 [RateLimiterTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/RateLimiterTest.java) 测试类，编写代码如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class RateLimiterTest {

    @Autowired
    private RedissonClient redissonClient;

    @Test
    public void test() throws InterruptedException {
        // 创建 RRateLimiter 对象
        RRateLimiter rateLimiter = redissonClient.getRateLimiter("myRateLimiter");
        // 初始化：最大流速 = 每 1 秒钟产生 2 个令牌
        rateLimiter.trySetRate(RateType.OVERALL, 2, 1, RateIntervalUnit.SECONDS);
//        rateLimiter.trySetRate(RateType.PER_CLIENT, 50, 1, RateIntervalUnit.MINUTES);

        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        for (int i = 0; i < 5; i++) {
            System.out.println(String.format("%s：获得锁结果(%s)", simpleDateFormat.format(new Date()),
                    rateLimiter.tryAcquire()));
            Thread.sleep(250L);
        }
    }

}
```

执行 `#test()` 测试用例，结果如下：

```
2019-10-02 22:46:40：获得锁结果(true)
2019-10-02 22:46:40：获得锁结果(true)
2019-10-02 22:46:41：获得锁结果(false)
2019-10-02 22:46:41：获得锁结果(false)
2019-10-02 22:46:41：获得锁结果(true)
```

- 第 1、2 次，成功获取锁。
- 第 3、4 次，因为每 1 秒产生 2 个令牌，所以被限流了。
- 第 5 次，已经过了 1 秒，所以获得令牌成功。

### 6.3.2 闲聊两句

有一点要纠正一下。Redisson 提供的限流器不是**严格且完整**的滑动窗口的限流器实现。举个例子，我们创建了一个每分钟允许 3 次操作的限流器。整个执行过程如下：

```
00:00:00 获得锁，剩余令牌 2 。
00:00:20 获得锁，剩余令牌 1 。
00:00:40 获得锁，剩余令牌 0 。
```

- 那么，00:01:00 时，锁的数量会恢复，按照 Redisson 的限流器来说。
- 如果是**严格且完整**的滑动窗口的限流器，此时在 00:01:00 剩余可获得的令牌数为 1 ，也就是说，起始点应该变成 00:00:20 。

如果基于 Redis **严格且完整**的滑动窗口的限流器，可以通过基于 Redis [Zset](http://redis.cn/commands.html#sorted_set) 实现。