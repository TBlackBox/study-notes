# 简介
在springboot 中,有时候我们需要打印出spring 容器管理了那些内，可以按照下面的配置看

# 配置方法

在springBoot启动类里面加入下面的代码即可
```java
@Bean
public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
    return args -> {
        System.out.println("Let's inspect the beans provided by Spring Boot:");

        String[] beanNames = ctx.getBeanDefinitionNames();
        Arrays.sort(beanNames);
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
    };
}
```