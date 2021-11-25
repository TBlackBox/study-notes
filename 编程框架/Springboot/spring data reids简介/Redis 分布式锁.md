## Redis 分布式锁

一说到分布式锁，大家一般会想到的就是基于 Zookeeper 或是 Redis 实现分布式锁。相对来说，在考虑性能为优先因素，不需要特别绝对可靠性的场景下，我们会优先考虑使用 Redis 实现的分布式锁。

在 Redisson 中，提供了 8 种分布式锁的实现，具体胖友可以看看 [《Redisson 文档 —— 分布式锁和同步器》](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器) 。真特码的强大！大多数开发者可能连 Redis 怎么实现分布式锁都没完全搞清楚，Redisson 直接给了 8 种锁，气人，简直了。

本小节，我们来编写一个简单使用 Redisson 提供的可重入锁 RLock 的示例。

创建 [LockTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-redisson/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/LockTest.java) 测试类，编写代码如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class LockTest {

    private static final String LOCK_KEY = "anylock";

    @Autowired // <1>
    private RedissonClient redissonClient;

    @Test
    public void test() throws InterruptedException {
        // <2.1> 启动一个线程 A ，去占有锁
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 加锁以后 10 秒钟自动解锁
                // 无需调用 unlock 方法手动解锁
                final RLock lock = redissonClient.getLock(LOCK_KEY);
                lock.lock(10, TimeUnit.SECONDS);
            }
        }).start();
        // <2.2> 简单 sleep 1 秒，保证线程 A 成功持有锁
        Thread.sleep(1000L);

        // <3> 尝试加锁，最多等待 100 秒，上锁以后 10 秒自动解锁
        System.out.println(String.format("准备开始获得锁时间：%s", new SimpleDateFormat("yyyy-MM-DD HH:mm:ss").format(new Date())));
        final RLock lock = redissonClient.getLock(LOCK_KEY);
        boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
        if (res) {
            System.out.println(String.format("实际获得锁时间：%s", new SimpleDateFormat("yyyy-MM-DD HH:mm:ss").format(new Date())));
        } else {
            System.out.println("加锁失败");
        }
    }

}
```

- 整个测试用例，意图是：1）启动一个线程 A ，先去持有锁 10 秒然后释放；2）主线程，也去尝试去持有锁，因为线程 A 目前正在占用着该锁，所以需要等待线程 A 释放到该锁，才能持有成功。
- `<1>` 处，注入 RedissonClient 对象。因为我们需要使用 Redisson 独有的功能，所以需要使用到它。
- `<2.1>` 处，启动线程 A ，然后调用 `RLock#lock(long leaseTime, TimeUnit unit)` 方法，加锁以后 10 秒钟自动解锁，无需调用 unlock 方法手动解锁。
- `<2.2>` 处，简单 sleep 1 秒，保证线程 A 成功持有锁。
- `<3>` 处，主线程，调用 `RLock#tryLock(long waitTime, long leaseTime, TimeUnit unit)` 方法，尝试加锁，最多等待 100 秒，上锁以后 10 秒自动解锁。

执行 `#test()` 测试用例，结果如下：

```
准备开始获得锁时间：2019-10-274 00:44:08
实际获得锁时间：2019-10-274 00:44:17
```

- 9 秒后（因为我们 sleep 了 1 秒），主线程成功获得到 Redis 分布式锁，符合预期。