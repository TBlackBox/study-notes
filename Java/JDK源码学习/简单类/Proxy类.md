# 简介

`Proxy`提供了创建动态代理类和实例的静态方法，它也是由这些方法创建的所有动态代理类的超类。

# newProxyInstance方法
```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
```
参数解释：
- loader表示类加载器
- interfaces表示代码要用来代理的接口 
-  h表示一个 InvocationHandler 对象

返回的是一个代理对象


# InvocationHandler 接口
每个代理的实例都有一个与之关联的 InvocationHandler 实现类，如果代理的方法被调用，那么代理便会通知和转发给内部的 InvocationHandler 实现类，由它决定处理。
```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```
InvocationHandler 内部只有一个 invoke() 方法，正是这个方法决定了怎么样处理代理传递过来的方法调用。其中参数proxy表示代理对象，method表示代理对象调用的方法，args表示调用的方法中的参数。所以Proxy动态产生的代理会调用InvocationHandler实现类，所以InvocationHandler才是实际执行者。