Redis 提供了 Pub/Sub 功能，实现简单的订阅功能，不了解的胖友，可以看看 [「Redis 文档 —— Pub/Sub」](http://redis.cn/topics/pubsub.html) 。

### 5.4.1 源码解析

暂时不提供，感兴趣的胖友，可以自己看看最核心的 [`org.springframework.data.redis.listener.RedisMessageListenerContainer`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/listener/RedisMessageListenerContainer.java) 类，Redis 消息监听器容器，基于 Pub/Sub 的 [SUBSCRIBE](http://redis.cn/commands/subscribe.html)、[PSUBSCRIBE](http://redis.cn/commands/psubscribe.html) 命令实现，我们只需要添加相应的 [`org.springframework.data.redis.connection.MessageListener`](https://github.com/spring-projects/spring-data-redis/blob/64b89137648f6c0e0c810c624e481bcfc0732f4e/src/main/java/org/springframework/data/redis/connection/MessageListener.java) 即可。不算复杂，1000 多行，只要调试下核心的功能即可。

### 5.4.2 具体示例

> 示例代码对应测试类：[PubSubTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/PubSubTest.java) 。

Spring Data Redis 实现 Pub/Sub 的示例，主要分成两部分：

- 配置 RedisMessageListenerContainer Bean 对象，并添加我们自己实现的 MessageListener 对象，用于监听处理相应的消息。
- 使用 RedisTemplate 发布消息。

下面，我们通过四个**小**步骤，来实现一个简单的示例。

**第一步，了解 Topic**

[`org.springframework.data.redis.listener.Topic`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/listener/Topic.java) 接口，表示 Redis 消息的 Topic 。它有两个子类实现：

- ChannelTopic ：对应 [SUBSCRIBE](http://redis.cn/commands/subscribe.html) 订阅命令。
- PatternTopic ：对应 [PSUBSCRIBE](http://redis.cn/commands/psubscribe.html) 订阅命令。

**第二步，实现 MessageListener 类**

创建 [TestChannelTopicMessageListener](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/main/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/listener/TestChannelTopicMessageListener.java) 类，编写代码如下：

```java
public class TestPatternTopicMessageListener implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] pattern) {
        System.out.println("收到 PatternTopic 消息：");
        System.out.println("线程编号：" + Thread.currentThread().getName());
        System.out.println("message：" + message);
        System.out.println("pattern：" + new String(pattern));
    }

}
```

- `message` 参数，可获得到具体的消息内容，不过是二进制数组，需要我们自己序列化。具体可以看下 [`org.springframework.data.redis.connection.DefaultMessage`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/connection/DefaultMessage.java) 类。
- `pattern` 参数，发布的 Topic 的内容。

有一点要注意，默认的 RedisMessageListenerContainer 情况下，MessageListener 是**并发**消费，在线程池中执行(具体见[传送门](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/listener/RedisMessageListenerContainer.java#L982-L988)代码)。所以如果想相同 MessageListener **串行**消费，可以在方法上加 `synchronized` 修饰，来实现同步。

**第三步，创建 RedisMessageListenerContainer Bean**

在 RedisConfiguration 中，配置 RedisMessageListenerContainer Bean 。代码如下：

```java
// RedisConfiguration.java

@Bean
public RedisMessageListenerContainer listenerContainer(RedisConnectionFactory factory) {
    // 创建 RedisMessageListenerContainer 对象
    RedisMessageListenerContainer container = new RedisMessageListenerContainer();

    // 设置 RedisConnection 工厂。😈 它就是实现多种 Java Redis 客户端接入的秘密工厂。感兴趣的胖友，可以自己去撸下。
    container.setConnectionFactory(factory);

    // 添加监听器
    container.addMessageListener(new TestChannelTopicMessageListener(), new ChannelTopic("TEST"));
//        container.addMessageListener(new TestChannelTopicMessageListener(), new ChannelTopic("AOTEMAN"));
//        container.addMessageListener(new TestPatternTopicMessageListener(), new PatternTopic("TEST"));
    return container;
}
```

要注意，虽然 RedisConnectionFactory 可以多次调用 [`#addMessageListener(MessageListener listener, Topic topic)`](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/listener/RedisMessageListenerContainer.java#L375-L396) 方法，但是一定要都是相同的 Topic 类型。例如说，添加了 ChannelTopic 类型，就不能添加 PatternTopic 类型。为什么呢？因为 RedisMessageListenerContainer 是基于**一次** [SUBSCRIBE](http://redis.cn/commands/subscribe.html) 或 [PSUBSCRIBE](http://redis.cn/commands/psubscribe.html) 命令，所以不支持**不同类型**的 Topic 。当然，如果是**相同类型**的 Topic ，多个 MessageListener 是支持的。

那么，可能会有胖友会问，如果我添加了 `"Test"` 给 MessageListener**A** ，`"AOTEMAN"` 给 MessageListener**B** ，两个 Topic 是怎么分发（Dispatch）的呢？在 RedisMessageListenerContainer 中，有个 [DispatchMessageListener](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/listener/RedisMessageListenerContainer.java#L961-L988) 分发器，负责将不同的 Topic 分发到配置的 MessageListener 中。看到此处，有木有想到 Spring MVC 的 DispatcherServlet 分发不同的请求到对应的 `@RequestMapping` 方法。

**第四步，使用 RedisTemplate 发布消息**

创建 [PubSubTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/PubSubTest.java) 测试类，编写代码如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PubSubTest {

    public static final String TOPIC = "TEST";

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            stringRedisTemplate.convertAndSend(TOPIC, "yunai:" + i);
            Thread.sleep(1000L);
        }
    }

}
```

- 通过 `RedisTemplate#convertAndSend(String channel, Object message)` 方法，PUBLISH 消息。

执行 `#test01()` 方法，运行结果如下：

```java
收到 ChannelTopic 消息：
线程编号：listenerContainer-2
message：yunai:0
pattern：TEST
收到 ChannelTopic 消息：
线程编号：listenerContainer-3
message：yunai:1
pattern：TEST
收到 ChannelTopic 消息：
线程编号：listenerContainer-4
message：yunai:2
pattern：TEST
```

- 整整齐齐，发送和订阅都成功了。注意，**线程编号**。

### 5.4.3 闲话两句

Redis 提供了 PUB/SUB 订阅功能，实际我们在使用时，一定要注意，它提供的**不是一个可靠的**订阅系统。例如说，有消息 PUBLISH 了，Redis Client 因为网络异常断开，无法订阅到这条消息。等到网络恢复后，Redis Client 重连上后，是无法获得到该消息的。相比来说，成熟的消息队列提供的订阅功能，因为消息会进行持久化（Redis 是不持久化 Publish 的消息的），并且有客户端的 ACK 机制做保障，所以即使网络断开重连，消息一样不会丢失。

> Redis 5.0 版本后，正式发布 Stream 功能，相信是有可能可以替代掉 Redis Pub/Sub 功能，提供可靠的消息订阅功能。

上述的场景，艿艿自己在使用 PUB/SUB 功能的时候，确实被这么坑过。当时我们的管理后台的权限，是缓存在 Java 进程当中，通过 Redis Pub/Sub 实现缓存的刷新。结果，当时某个 Java 节点网络出问题，恰好那个时候，有一条刷新权限缓存的消息 PUBLISH 出来，结果没刷新到。结果呢，运营在访问某个功能的时候，一会有权限（因为其他 Java 节点缓存刷新了），一会没有权限。

最近，艿艿又去找了几个朋友请教了下，问问他们在生产环境下，是否使用 Redis Pub/Sub 功能，他们说使用 Kafka、或者 RocketMQ 的广播消费功能，更加可靠有保障。

对了，我们有个管理系统里面有 Websocket 需要实时推送管理员消息，因为不知道管理员当前连接的是哪个 Websocket 服务节点，所以我们是通过 Redis Pub/Sub 功能，广播给所有 Websocket 节点，然后每个 Websocket 节点判断当前管理员是否连接的是它，如果是，则进行 Websocket 推送。因为之前网络偶尔出故障，会存在消息丢失，所以近期我们替换成了 RocketMQ 的广播消费，替代 Redis Pub/Sub 功能。

当然，不能说 Redis Pub/Sub 毫无使用的场景，以下艿艿来列举几个：

- 1、在使用 Redis Sentinel 做高可用时，Jedis 通过 Redis Pub/Sub 功能，实现对 Redis 主节点的故障切换，刷新 Jedis 客户端的主节点的缓存。如果出现 Redis Connection 订阅的异常断开，会重新**主动**去 Redis Sentinel 的最新主节点信息，从而解决 Redis Pub/Sub 可能因为网络问题，丢失消息。
- 2、Redis Sentinel 节点之间的部分信息同步，通过 Redis Pub/Sub 订阅发布。
- 3、在我们实现 Redis 分布式锁时，如果获取不到锁，可以通过 Redis 的 Pub/Sub 订阅锁释放消息，从而实现其它获得不到锁的线程，快速抢占锁。当然，Redis Client 释放锁时，需要 PUBLISH 一条释放锁的消息。在 Redisson 实现分布式锁的源码中，我们可以看到。
- 4、Dubbo 使用 Redis 作为注册中心时，使用 Redis Pub/Sub 实现注册信息的同步。

也就是说，如果想要有保障的使用 Redis Pub/Sub 功能，需要处理下发起订阅的 Redis Connection 的异常，例如说网络异常。然后，重新主动去查询最新的数据的状态。😈