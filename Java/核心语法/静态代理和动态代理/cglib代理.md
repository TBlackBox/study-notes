# cglib代理
cglib (Code Generation Library )是一个第三方代码生成类库，运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展。

## cglib特点
JDK的动态代理有一个限制，就是使用动态代理的对象必须实现一个或多个接口。
如果想代理没有实现接口的类，就可以使用CGLIB实现。
- CGLIB是一个强大的高性能的代码生成包，它可以在运行期扩展Java类与实现Java接口。
它广泛的被许多AOP的框架使用，例如`Spring AOP`和`dynaop`，为他们提供方法的interception（拦截）。
- CGLIB包的底层是通过使用一个小而快的字节码处理框架`ASM`，来转换字节码并生成新的类。
不鼓励直接使用ASM，因为它需要你对JVM内部结构包括class文件的格式和指令集都很熟悉。

## cglib与动态代理最大的区别

1. 使用动态代理的对象必须实现一个或多个接口
2. 使用cglib代理的对象则无需实现接口，达到代理类无侵入。
3. 使用cglib需要引入cglib的jar包，如果你已经有spring-core的jar包，则无需引入，因为spring中包含了cglib。

## cglib的Maven坐标
```
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.5</version>
</dependency>
```
## 例子
举例：用户跳舞例子实现
### 穿件类
```
public class UserServiceImpl{
	public void dance() {
		System.out.println("本人正在跳舞");
	}
}
```

### 创建代理对象
```
public class ProxyFactory implements MethodInterceptor{

    private Object target;//维护一个目标对象
    public ProxyFactory(Object target) {
        this.target = target;
    }
    
    //为目标对象生成代理对象
    public Object getProxyInstance() {
        //工具类
        Enhancer en = new Enhancer();
        //设置父类
        en.setSuperclass(target.getClass());
        //设置回调函数
        en.setCallback(this);
        //创建子类对象代理
        return en.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("去跳舞的路上");
        // 执行目标对象的方法
        Object returnValue = method.invoke(target, args);
        System.out.println("跳舞完回家的路上");
        return null;
    }
}
```
## 调用方式
```
public static void main(String[] args) {
		UserServiceImpl userServiceImpl = new UserServiceImpl();
		UserServiceImpl proxy = (UserServiceImpl) new ProxyFactory(userServiceImpl).getProxyInstance();
		proxy.dance();
}
```

### 输出
```
去跳舞的路上
本人正在跳舞
跳舞完回家的路上
```