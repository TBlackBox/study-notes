# 简介

 Redis命令十分丰富，包括的命令组有Cluster、Connection、Geo、Hashes、HyperLogLog、Keys、Lists、Pub/Sub、Scripting、Server、Sets、Sorted Sets、Strings、Transactions一共14个redis命令组两百多个redis命令。



[Redis 命令参考 — Redis 命令参考 (redisfans.com)](http://doc.redisfans.com/)

[Redis命令中心（Redis commands） -- Redis中国用户组（CRUG）](http://www.redis.cn/commands.html#hash)



# Cluster



# Connection

1. AUTH

   验证服务器命令

   ```
   auth password
   ```

2. ECHO

   回显输入的字符串

   ```
   echo message
   ```

3. PING

   ping 服务器

   ```
   ping 
   ```

4. QUIT

   关闭连接，退出

5. SELECT

   选择新的数据库

   ```
   select index
   ```

6. SWAPDB

   交换redis 的两个数据库

   ```
   swapdb index index
   ```

# Geo





# Pub/Sub

# PSUBSCRIBE

**PSUBSCRIBE pattern [pattern ...]**

订阅一个或多个符合给定模式的频道。

每个模式以 `*` 作为匹配符，比如 `it*` 匹配所有以 `it` 开头的频道( `it.news` 、 `it.blog` 、 `it.tweets` 等等)， `news.*` 匹配所有以 `news.` 开头的频道( `news.it` 、 `news.global.today` 等等)，诸如此类。

# PUBLISH

**PUBLISH channel message**

将信息 `message` 发送到指定的频道 `channel` 。



**PUBSUB <subcommand> [argument [argument ...]]**

[*PUBSUB*](http://doc.redisfans.com/pub_sub/pubsub.html#pubsub) 是一个查看订阅与发布系统状态的内省命令， 它由数个不同格式的子命令组成， 以下将分别对这些子命令进行介绍。



# PUNSUBSCRIBE

**PUNSUBSCRIBE [pattern [pattern ...]]**

指示客户端退订所有给定模式。

如果没有模式被指定，也即是，一个无参数的 `PUNSUBSCRIBE` 调用被执行，那么客户端使用 [*PSUBSCRIBE*](http://doc.redisfans.com/pub_sub/psubscribe.html#psubscribe) 命令订阅的所有模式都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的模式。



# SUBSCRIBE

**SUBSCRIBE channel [channel ...]**

订阅给定的一个或多个频道的信息。

# UNSUBSCRIBE

**UNSUBSCRIBE [channel [channel ...]]**

指示客户端退订给定的频道。

如果没有频道被指定，也即是，一个无参数的 `UNSUBSCRIBE` 调用被执行，那么客户端使用 [*SUBSCRIBE*](http://doc.redisfans.com/pub_sub/subscribe.html#subscribe) 命令订阅的所有频道都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的频道。