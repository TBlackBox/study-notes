

## 唯一规则的硬性要求

1. 全局唯一
2. 趋势递增
3. 单调递增
4. 信息安全
5. 含时间戳





## 为什么UUID会导致入库性能变差

1. 无序，无法预测他的生成顺序，不能生成递增的有序数列。
2. 主建，ID作为主键时在特定的环境中会存在一些问题。
3. 索引，B+树索引的分裂。





## 唯一ID 的几种方式

1. UUID

   只有唯一性，趋势递增

2. MYSQL（insert 或者 resplace into）

   唯一性，递增,存在性能问题（需要集群 太耗了）。

3. redis 生成

   - 单价版

     reidis 是单线程 天生保证原子性，可以使用incr 和incrby 实现

   - 集群版

需要设置步长，号不连续。花费的精力也比较大。

4. 雪花算法产生的ID



# 雪花算法

![image-20210224114722970](../../\sources\img\image-20210224114722970.png)

<img src="../../\sources\img\image-20210224115002518.png" alt="image-20210224115002518"  />

![image-20210224115158838](../../\sources\img\image-20210224115158838.png)



![image-20210224121808372](../../\sources\img\image-20210224121808372.png)

## 通过hutool工具包获取

http://hutool.com



## 引入方法

1. 添加依赖包

   ```xml
   <dependency>
       <groupId>cn.hutool</groupId>
       <artifactId>hutool-all</artifactId>
       <version>5.3.10</version>
   </dependency>
   
   ```

   

2. 工具类

   ```java
   import cn.hutool.core.lang.Snowflake;
   import cn.hutool.core.net.NetUtil;
   import cn.hutool.core.util.IdUtil;
   import com.fasterxml.jackson.annotation.JsonFormat;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.stereotype.Component;
   import javax.annotation.PostConstruct;
   @Component
   @Slf4j
   public class SnowflakeConfig {
       @JsonFormat(shape = JsonFormat.Shape.STRING)
       private long workerId = 0;//为终端ID
       private long datacenterId = 1;//数据中心ID
       private Snowflake snowflake = IdUtil.createSnowflake(workerId,datacenterId);
       @PostConstruct
       public void init(){
           workerId = NetUtil.ipv4ToLong(NetUtil.getLocalhostStr());
           log.info("当前机器的workId:{}",workerId);
       }
       public synchronized long snowflakeId(){
           return snowflake.nextId();
       }
       public synchronized long snowflakeId(long workerId,long datacenterId){
           Snowflake snowflake = IdUtil.createSnowflake(workerId, datacenterId);
           return snowflake.nextId();
       }
   }
   
   
   ```

   





















































































































































































































































































