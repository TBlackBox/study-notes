# 简介
SpringBoot集成了amqp协议实现的RabbitMq,我们只需要按照SpringBoot配置其他模块的步骤一样，配置RabbitMq就可以了，可以参照RabbitAutoConfiguration自动配置类进行配置。

# SpringBoot集成RabbitMq
1. 引入spring-boot-starter-amqp 启动器
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

2. 在配置文件里面培训RabbitMq相关信息
```java
spring:
  rabbitmq:
    host: 192.168.0.110
    username: guest
    password: guest
    #默认端口 5672
    port: 5672
```

3. 通个RabbitTemplate进行发消息和收消息
```java
@Autowired
RabbitTemplate rabbitTemplate;


@Test
public void testRabbitMqSendMsg(){
    //发送消息  message 需要封装，定义消息体体和消息头
    //exchange 交换器名字
    //routingKey  路由可
    //message 消息
    //rabbitTemplate.send(exchange, routingKey, message);

    //发送红简易消息，不需要封装消息，object 默认当成消息体，发送给RabbitMq
    //发送的消息默认用Java序列化的方式存储到RabbitMq队列中（当然可以改序列化的方式）
    //rabbitTemplate.convertAndSend(exchange, routingKey, object);
}
```

4. 接受消息
```java
@Autowired
RabbitTemplate rabbitTemplate;

@Test
public void testRabbitRecMsg(){
    //通过receive 接受消息  message 是个消息体  需要解析
    //Message message = rabbitTemplate.receive(queueName);
    //message 队列名字  接到的消息是object  消息对象
    //Object o = rabbitTemplate.convertSendAndReceive(exchange, routingKey, message);
}
```
5. 序列化为json 存储
在RabbitTemplate里面有个消息转化器SimpleMessageConverter,默认的是使用的jdk的序列化方式。只需要更换转换器就可以了，注意，这里可能不同的SpringBoot版本不同，有所区别，参照RabbitTemplate源码里面的信息配置
```java
@Configuration
public class MyAMQPConfig {

    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

6. 通过注解的形式监听消息
+ 在主入口上 开启基于注解的RabbitMq模式
```java 
@EnableRabbit  //开启基于注解的RabbitMq模式
@SpringBootApplication
public class SpringbootApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringbootApplication.class, args);
	}
}
```

+ 通过  @RabbitListener注解监听消息
```java
@Service
public class RabbitServer {

    //直接结算消息
    //里面可以指定队列的名字等参数
    @RabbitListener(queues="queuesName")
    public void receive(User user){
        //接受消息
        System.out.println("接受到的 消息"+user);
    }

    //接受消息体
    @RabbitListener(queues="queuesName")
    public void receive(Message message){
        System.out.println("消息体："+message);
    }
}
```

7. 通过AmqpAdmin管理RabbitMq队列，交换器，绑定等内容
SpringBoot在RabbitAutoConfiguration类里面里注入了AmqpAdmin类，方便我们对RabbitMq的后端进行管理,下面列举简单的用法，详细用法可以参照接口测试 。
```java
@Autowired
AmqpAdmin amqpAdmin;

@Test
public void contextLoads() {
    //新建交换器
    amqpAdmin.declareExchange(new DirectExchange("springBoot.directExchange"));
    amqpAdmin.declareExchange(new FanoutExchange("springBoot.fanoutExchange"));
    amqpAdmin.declareExchange(new TopicExchange("springBoot.topicExchange"));
    //新建队列
    amqpAdmin.declareQueue(new Queue("springBoot.queue", true));
    //绑定
    amqpAdmin.declareBinding(new Binding("springBoot.queue", Binding.DestinationType.QUEUE, "springBoot.fanoutExchange", "", null));
    amqpAdmin.declareBinding(new Binding("springBoot.queue", Binding.DestinationType.QUEUE, "springBoot.topicExchange", "springBoot.*", null));
    amqpAdmin.declareBinding(new Binding("springBoot.queue", Binding.DestinationType.QUEUE, "springBoot.directExchange", "", null));
}

@Test
public void remove() {
    //移除交换器
    amqpAdmin.deleteExchange("springBoot.directExchange");
    amqpAdmin.deleteExchange("springBoot.fanoutExchange");
    amqpAdmin.deleteExchange("springBoot.topicExchange");
    //移除队列
    amqpAdmin.deleteQueue("springBoot.queue");
}
```
