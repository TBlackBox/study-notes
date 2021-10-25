## Script

Redis 提供 Lua 脚本，满足我们希望组合排列使用 Redis 的命令，保证**串行**执行的过程中，不存在并发的问题。同时，通过将多个命令组合在同一个 Lua 脚本中，一次请求，直接处理，也是一个提升性能的手段。不了解的胖友，可以看看 [「Redis 文档 —— Lua 脚本」](http://redis.cn/documentation.html) 。

下面，我们来看看 Spring Data Redis 使用 Lua 脚本的示例。

> 示例代码对应测试类：[ScriptTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/ScriptTest.java) 。

**第一步，编写 Lua 脚本**

创建 `resources/compareAndSet.lua` 脚本，实现 CAS 功能。代码如下：

```lua
if redis.call('GET', KEYS[1]) ~= ARGV[1] then
    return 0
end
redis.call('SET', KEYS[1], ARGV[2])
return 1
```

- 第 1 到 3 行：判断 `KEYS[1]` 对应的 VALUE 是否为 `ARGV[1]` 值。如果不是（Lua 中不等于使用 `~=`），则直接返回 0 表示失败。
- 第 4 到 5 行：设置 `KEYS[1]` 对应的 VALUE 为新值 `ARGV[2]` ，并返回 1 表示成功。

**第二步，调用 Lua 脚本**

创建 [ScriptTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-11-spring-data-redis/lab-07-spring-data-redis-with-jedis/src/test/java/cn/iocoder/springboot/labs/lab10/springdatarediswithjedis/ScriptTest.java) 测试类，编写代码如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ScriptTest {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() throws IOException {
        // <1.1> 读取 /resources/lua/compareAndSet.lua 脚本 。注意，需要引入下 commons-io 依赖。
        String  scriptContents = IOUtils.toString(getClass().getResourceAsStream("/lua/compareAndSet.lua"), "UTF-8");
        // <1.2> 创建 RedisScript 对象
        RedisScript<Long> script = new DefaultRedisScript<>(scriptContents, Long.class);
        // <2> 执行 LUA 脚本
        Long result = stringRedisTemplate.execute(script, Collections.singletonList("yunai:1"), "shuai02", "shuai");
        System.out.println(result);
    }
}
```

- `<1.1>` 行，读取 `/resources/lua/compareAndSet.lua` 脚本。注意，需要引入下 `commons-io` 依赖。
- `<1.2>` 行，创建 DefaultRedisScript 对象。第一个参数是脚本内容( `scriptSource` )，第二个是脚本执行返回值( `resultType` )。
- `<2>` 处，调用 [`RedisTemplate#execute(RedisScript script, List keys, Object... args)`](https://github.com/spring-projects/spring-data-redis/blob/64b89137648f6c0e0c810c624e481bcfc0732f4e/src/main/java/org/springframework/data/redis/core/RedisTemplate.java#L342-L360) 方法，发送 Redis 执行 LUA 脚本。