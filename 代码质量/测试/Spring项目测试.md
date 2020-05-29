# spring 项目测试

# 引入相应依赖
使用spring的测试框架需要加入以下依赖包：
JUnit 4 （官方下载：http://www.junit.org/）
Spring Test （Spring框架中的test包）
Spring 相关其他依赖包（不再赘述了，就是context等包）
```
 <dependency>
     <groupId>junit</groupId>
     <artifactId>junit</artifactId>
     <version>4.13</version>
     <scope>test</scope>
 </dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.6.RELEASE</version>
    <scope>test</scope>
</dependency>
```
## 单元测试案列
需要依赖SpringJUnit4ClassRunner（需要引入spring-test依赖）。
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {
        "classpath:applicationContext.xml",
})
public class SpringTest {
    @Resource
    private UserDao userDao;
    
    @Test
    public void testAdd(){
        User u = userDao.add(new User("张三"));
        
        Assert.assertTrue(u.getId() > 0);        
    }
}

```

## 几个注解说明
- @RunWith：用于指定junit运行环境，是junit提供给其他框架测试环境接口扩展，为了便于使用spring的依赖注入，spring提供了org.springframework.test.context.junit4.SpringJUnit4ClassRunner作为Junit测试环境

-  @ContextConfiguration({"classpath:applicationContext.xml","classpath:spring/buyer/applicationContext-service.xml"}) 
导入配置文件，这里我的applicationContext配置文件是根据模块来分类的。如果有多个模块就引入多个“applicationContext-service.xml”文件。如果所有的都是写在“applicationContext.xml”中则这样导入： 
@ContextConfiguration(locations = "classpath:applicationContext.xml") 


- @TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)这里的事务关联到配置文件中的事务控制器（transactionManager = "transactionManager"），同时指定自动回滚（defaultRollback = true）。这样做操作的数据才不会污染数据库！ 
- @Transactional:这个非常关键，如果不加入这个注解配置，事务控制就会完全失效！ 

AbstractTransactionalDataSourceSpringContextTests要想构建这一系列的无污染纯绿色事务测试框架就必须找到这个基类！（即所有事务均不生效）

 