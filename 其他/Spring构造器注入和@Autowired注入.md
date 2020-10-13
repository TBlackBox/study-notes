# 简介

因为业务关系也看了些同事写的代码，因为公司没有明确规定，有一部分人在注入依赖的时候使用了用Spring推荐的构造器注入的方式，一部分人使用了@Autowired的注解进行注入。

因此，接下来我试着总结归纳一下相关的情况，做一次回顾。

用自己的话去说出来才算是真的掌握。

# 使用介绍

## 1.@Autowired注入

 

```java
@Service
public class AdminService extends AbstractService<Admin, Integer>{
	@Autowired
	UserRepository userRepository;
}
```

## 2.构造器注入

Spring4.3+之后，constructor注入支持非显示注入方式。

```java
@Service
public class AdminServiceImpl implements AdminService {

    private final PostService postService;


    public AdminServiceImpl(PostService postService){
        this.postService = postService;
    }
```

##  3.setter注入

```java
@RestController



@RequestMapping("/test")
public class TestController {

    private final List<TestService> testServices;

    private final List<ChainAsbtract> chains;

    @Autowired
    public void setTestServices(List<TestService> testServices){

        this.testServices = testServices;
    }

    @Autowired
    public void setTestServices(List<ChainAsbtract> chains){
        this.chains = chains;
    }

}
```

事实上，spring在4.x版本后就推荐使用构造器的方式的来注入fileld

> **官方推荐理由**
>
> 单一职责: 当使用构造函数注入的时候，你会很容易发现参数是否过多，这个时候需要考虑你这个类的职责是否过大，考虑拆分的问题；而当使用@Autowired注入field的时候，不容易发现问题
>
> 依赖不可变: 只有使用构造函数注入才能注入final
>
> 依赖隐藏:使用依赖注入容器意味着类不再对依赖对象负责，获取依赖对象的职责就从类抽离出来，IOC容器会帮你自动装备。这意味着它应该使用更明确清晰的公用接口方法或者构造器，这种方式就能很清晰的知道类需要什么和到底是使用setter还是构造器
>
> 降低容器耦合度: 依赖注入框架的核心思想之一是托管类不应依赖于所使用的DI容器。换句话说，它应该只是一个普通的POJO，只要您将其传递给所有必需的依赖项，就可以独立地实例化。这样，您可以在单元测试中实例化它，而无需启动IOC容器并单独进行测试（使用一个可以进行集成测试的容器）。如果没有容器耦合，则可以将该类用作托管或非托管类，甚至可以切换到新的DI框架。

 另外，在使用构造器的使用能避免注入的依赖是空的情况。