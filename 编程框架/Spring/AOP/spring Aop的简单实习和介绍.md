# 简介
我们在保存一个用户的时候,需要在保存的方法前开启事务,保存完后,提交事务,要实现这个功能,可以有4种方法
- 在每个保存方法前后直接写
- 通过静态代理
- 通过动态代理
- 通过cglib代理
具体做法这里都不描述了，不晓得看静态代理后动态代理的相关知识。
下面通过spring aop 来实现相关功能，像上面说的样，开启事务在保存方法前做的，提交事务在保持方法后面做的，在aop里面，在方法之前的叫`Before Advice（前置增强）`,方法后面的叫`After Advice（后置增强）`，把这两个合并起来就叫` Around Advice（环绕增强）`. 

我们只需要去实现这些所谓的“增强类”，让他们横切到代码中，而不是将这些写死在代码中。

# 通过编码的形式实现前置增强、后置增强、环绕增强

## 创建一个实现调用方法
```
public class UserDao {
	public void save() {
		System.out.println("保存对象");
	}
}
```

## 前置增强类
需要实现`org.springframework.aop.MethodBeforeAdvice`接口
```
import java.lang.reflect.Method;
import org.springframework.aop.MethodBeforeAdvice;
//前置增强类
public class BeforeAdvice implements MethodBeforeAdvice{

	@Override
	public void before(Method method, Object[] args, Object target) throws Throwable {
		System.out.println("开启事务");
	}

}
```
## 后置增强类
需要实现`org.springframework.aop.AfterReturningAdvice`接口
```
import java.lang.reflect.Method;
import org.springframework.aop.AfterReturningAdvice;

//后置增强类
public class AfterAdvice implements AfterReturningAdvice{

	@Override
	public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
		System.out.println("关闭事务");
	}
}
```
## 这里前置后置用一个类实现
```
import java.lang.reflect.Method;
import org.springframework.aop.AfterReturningAdvice;
import org.springframework.aop.MethodBeforeAdvice;

public class BeforeAndAfterAdvice implements MethodBeforeAdvice,AfterReturningAdvice{

	@Override
	public void before(Method method, Object[] args, Object target) throws Throwable {
		System.out.println("开启事务 ");
	}
	
	@Override
	public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
		System.out.println("关闭事务");
	}
}
```

## 环绕增强
需要实现`org.aopalliance.intercept.MethodInterceptor`接口
```
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;

public class AroundAdvice implements MethodInterceptor{

	@Override
	public Object invoke(MethodInvocation invocation) throws Throwable {
		System.out.println("开启事务");

        Object result = invocation.proceed();
        
		System.out.println("关闭事务");
		return null;
	}
}
```

## 调用（编码的形式）
```
public static void main(String[] args) {
    // 创建代理工厂
    ProxyFactory proxyFactory = new ProxyFactory();     
    // 射入目标类对象 如果是接口的话  那这里介绍接口对应的实现
    proxyFactory.setTarget(new UserDao()); 

    /*
    * 添加各种增强
    */
    //1 添加前置增强
    proxyFactory.addAdvice(new BeforeAdvice());
    //2 添加后置增强 
    proxyFactory.addAdvice(new AfterAdvice());  

    // 从代理工厂中获取代理
    UserDao userDao = (UserDao) proxyFactory.getProxy();
    // 调用代理的方法
    userDao.save();                              
}
```
** 说明： **
上面分别调用了前置和后置增强，如果只想用一个，就调一个就可以了。前置后置用一个类实现的调用的话把两个增强改为
```
proxyFactory.addAdvice(new BeforeAndAfterAdvice());
```
同理环绕增强换成下面的即可：
```
proxyFactory.addAdvice(new AroundAdvice());
```

# 通过声明的方式实现 前置增强、后置增强、环绕增强
前面通过编码的形式实现了3个增强，下面通过配置文件声明的形式实现

同样的例子，只是上面的是直接实现，为了方便区别，下面通过接口和实现的方式实现上面的例子
## 创建接口和实现类
接口
```
public interface IUserDao {
	public void save();
}
```
实现
```
@Component
public class UserDaoImpl implements IUserDao{

	@Override
	public void save() {
		System.out.println("保存用户");
	}
}
```

## 添加配置文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

  	 <!-- 扫描类 -->
    <context:component-scan base-package="aop"/>

    <!-- 配置一个代理 -->
    <bean id="userDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="interfaces" value="aop.IUserDao"/> <!-- 需要代理的接口 -->
        <property name="target" ref="userDaoImpl"/>       <!-- 接口实现类 -->
        <property name="interceptorNames">                 <!-- 拦截器名称（也就是增强类名称，Spring Bean 的 id） -->
            <list>
                <value>stateAroundAdvice</value>
            </list>
        </property>
        <!-- 如果只有一个增强的话 可以直接这样写  -->
        <!-- <property name="interceptorNames" value="stateAroundAdvice"></property> -->
    </bean>
    
</beans>
```
## 调用
```
public static void main(String[] args) {
		// 获取 Spring Context
		ApplicationContext context = new ClassPathXmlApplicationContext("/applicationContext.xml"); 
		// 从 Context 中根据 id 获取 Bean 对象（其实就是一个代理）
		IUserDao userDao = (IUserDao) context.getBean("userDaoProxy");                       
        // 调用代理的方法
        userDao.save();                                                              
}
```

## 输出
```
开启事务
保存用户
关闭事务
```

优点：代码量确实少了，我们将配置性的代码放入配置文件，这样也有助于后期维护。更重要的是，代码只关注于业务逻辑，而将配置放入文件中。这是一条最佳实践！