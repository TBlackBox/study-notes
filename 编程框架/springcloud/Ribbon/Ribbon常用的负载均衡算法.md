# ribbon常用的负载均衡算法

IRunle接口，Ribbon使用该接口，根据特定的算法从所有服务中，选择一个服务。

```java
public interface IRule {
    Server choose(Object var1);

    void setLoadBalancer(ILoadBalancer var1);

    ILoadBalancer getLoadBalancer();
}
```





![image-20210218120427990](../../../sources\springcloud\image-20210218120427990.png)

#### IRule接口又7个实现类，每个类代表一个负载均衡算法

都在`com.netflix.loadbalancer`包下

|  类  | 规则|
|-------|--------|
|RoundRobinRule | 轮询|
|RandomRule|随机|
|RetryRule|先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试|
|WeightedResponseTimeRule|对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择|
|BestAvailableRule|会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务|
|AvailabilityFilteringRule|先过滤掉故障实例，再选择并发较小的实例|
|ZoneAvoidanceRule|默认规则，复合判断server所在区域的性能和server的可用性选择服务器|

