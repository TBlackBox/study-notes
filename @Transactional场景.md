# @Transactional 失效的情况

事务有编程式事务和声明式事务两种

- 编程式事务：在代码中手动处理事务的提交，回滚等等，代码侵入性比较强。
- 声明式事务：基于AOP切面，代码侵入性比较低，实际开发中用得也比较多。声明式事务也有两种实现方式，是基于TX和AOP的xml配置文件方式，二种就是基于@Transactional注解了。

# @Transactional介绍

该注解可以作用余接口，类，类方法。

- **作用于类**：当把@Transactional 注解放在类上时，表示所有该类的public方法都配置相同的事务属性信息。
- **作用于方法**：当类配置了@Transactional，方法也配置了@Transactional，方法的事务会覆盖类的事务配置信息。
- **作用于接口**：不推荐这种使用方法，因为一旦标注在Interface上并且配置了Spring AOP 使用CGLib动态代理，将会导致@Transactional注解失效

## 属性

**propagation属性**

propagation 代表事务的传播行为，默认值为 Propagation.REQUIRED，其他的属性信息如下：

- Propagation.REQUIRED：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。**(** 也就是说如果A方法和B方法都添加了注解，在默认传播模式下，A方法内部调用B方法，会把两个方法的事务合并为一个事务 **）**
- Propagation.SUPPORTS：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
-  Propagation.MANDATORY：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
- Propagation.REQUIRES_NEW：重新创建一个新的事务，如果当前存在事务，暂停当前的事务。**(** 当类A中的 a 方法用默认Propagation.REQUIRED模式，类B中的 b方法加上采用 Propagation.REQUIRES_NEW模式，然后在 a 方法中调用 b方法操作数据库，然而 a方法抛出异常后，b方法并没有进行回滚，因为Propagation.REQUIRES_NEW会暂停 a方法的事务 **)**
- Propagation.NOT_SUPPORTED：以非事务的方式运行，如果当前存在事务，暂停当前的事务。
- Propagation.NEVER：以非事务的方式运行，如果当前存在事务，则抛出异常。
- Propagation.NESTED ：和 Propagation.REQUIRED 效果一样。

**isolation 属性**

isolation ：事务的隔离级别，默认值为Isolation.DEFAULT

- solation.DEFAULT：使用底层数据库默认的隔离级别。
- Isolation.READ_UNCOMMITTED
- Isolation.READ_COMMITTED
- Isolation.REPEATABLE_READIsolation.SERIALIZABLE

**timeout 属性**

timeout ：事务的超时时间，默认值为 -1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

**readOnly 属性**

readOnly ：指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。

**rollbackFor 属性**

rollbackFor ：用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

**noRollbackFor属性\****

noRollbackFor：抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。

## 失效场景

1. **@Transactional 应用在非 public 修饰的方法上**

主要是spring aop 有个方法，不是public 的  不会获取@Transactional的属性配置信息

**2、@Transactional 注解属性 propagation 设置错误**

这种失效是由于配置错误，若是错误的配置以下三种 propagation，事务将不会发生回滚。

- TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

- TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。

- TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

3. **@Transactional 注解属性 rollbackFor 设置错误**

   rollbackFor 可以指定能够触发事务回滚的异常类型。Spring默认抛出了未检查unchecked异常（继承自 RuntimeException 的异常）或者 Error才回滚事务；其他异常不会触发回滚事务。如果在事务中抛出其他类型的异常，但却期望 Spring 能够回滚事务，就需要指定 **rollbackFor**属性。

4. **、同一个类中方法调用，导致@Transactional失效**

   开发中避免不了会对同一个类里面的方法调用，比如有一个类Test，它的一个方法A，A再调用本类的方法B（不论方法B是用public还是private修饰），但方法A没有声明注解事务，而B方法有。则外部调用方法A之后，方法B的事务是不会起作用的。这也是经常犯错误的一个地方。

5. **异常被你的 catch“吃了”导致@Transactional失效**

   这种情况是最常见的一种@Transactional注解失效场景，也就是注解了@Transactional的方法类，把异常给捕获了。

6. **数据库引擎不支持事务**

   这种情况出现的概率并不高，事务能否生效数据库引擎是否支持事务是关键。常用的MySQL数据库默认使用支持事务的innodb引擎。一旦数据库引擎切换成不支持事务的myisam，那事务就从根本上失效了。