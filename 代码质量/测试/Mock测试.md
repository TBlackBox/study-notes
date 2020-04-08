
# Mork测试的简介
在写单元测试时会经常碰到一些依赖于外部环境的场景，比如数据库查询、网络请求。有时候，这些外部依赖并非是一直可用的或者暂时没有开发完成的，经常会对测试产生影响。而Mock测试工具则是为了解决这个问题发明的。Java中常用的Mock测试框架主要是EasyMock和PowerMock。这里的Mock就是打桩（Stub）或者模拟，当你调用一个不好在测试中创建的对象时，Mock框架为你模拟一个和真实对象类似的替身来完成相应的行为。

# EasyMork

以EasyMock的使用举例，来看看如何使用Mock框架来写单元测试。

有一个IAppService的接口实现，依赖于AppDao这个接口。
```
public AppServiceImpl implements IAppService{
    private AppDao appDao;
    
    public App getAppById(long id){
        return appDao.getById(id);
    }
    
    public void updateAppById(long id){
        return appDao.update(app);
    }
    
    public void setAppDao(AppDao appDao){
        this.appDao = appDao;
    }
}
```
如果这时候AppDao并没有实现，那么如果对AppService做单元测试，就需要Mock出一个AppDao。
```
public class AppServiceImplTest {
    @Test
    public void testGet() {
        App expectedApp= new App();
        app.setId(1);
        AppDao mock  = EasyMock.createMock(AppDao.class);//创建Mock对象                         
        Easymock.expect(mock.getById(1)).andReturn(expectedApp);//录制Mock对象预期行为
        Easymock.replay(mock); //重放Mock对象，测试时以录制的对象预期行为代替真实对象的行为
        
        AppServiceImpl  service = new AppServiceImpl();
        service.setAppDao(mock);
        App app = service.getAppById(1);//调用测试方法
        assertEquals(expectedApp, App); //断言测试结果
        
        Easymock.verify(mock); //验证Mock对象被调用
    }
}
```
这里需要注意的是：在EasyMock3.0之前，其使用JDK的动态代理实现Mock对象创建，因此只能针对接口进行Mock。3.0后则加入了使用CGLIB的实现方式，因此支持对普通类的Mock。

以上是一个简单的测试，除此之外，还有以下常用场景：

+ 基本参数匹配,即对所有匹配的参数，都做Mock。
```
Easymock.expect(mock.getById(Easymock.isA(Long.class))).andReturn(exceptedApp); //这样无论传入的id是多少，都会返回这个expectedApp而不是发生Unexpect method的Mock异常。
类似于EasyMock.isA的还有anyInt()，anyObject()， isNull()，same(), startsWith()等。
```
+ AppServiceImpl在调用AppDao的getById方法时发生异常
```
Easymock.expect(mock.getById(1)).andThrow(new RuntimeException());
```
+ void方法Mock
```
mock.updateAppById(1);
EasyMock.expectLastCall().anytimes();
```
使用expectLastCall方法即可。

+ 多次调用返回不同值的Mock。

此种业务场景的列子就是iterator的hasNext方法，如果一直返回true,那么代码会陷入无限循环。正常的测试逻辑应该是先返回几次true执行循环体，然后再返回false退出循环。
```
EasyMock.expect(rs.next()).andReturn(true).times(2).andReturn(false).times(1);
```
上述使用EasyMock已经可以解决大部分测试需求，但由于EasyMock使用JDK动态代理以及CGLIB实现子类的方式，其对于静态、final类型的类和方法以及私有方法是无法完成mock的。PowerMock则在EasyMock的基础上进行了扩展，使用字节码操作技术直接对生成的字节码类文件进行修改，从而能够对静态、final类型的类和方法、私有方法进行Mock，还可以对类进行部分Mock。


# PowerMock   

PowerMock的工作过程和EasyMock类似，不同之处在于需要在类层次声明@RunWith(PowerMockRunner.class)注解，以确保使用PowerMock框架引擎执行单元测试。

+ 静态方法
```
// Util.java
public class Util{
  public static String getEnv(String name){
      return System.getenv(name);
  }
}
   
//UtilTest.java
@RunWith(PowerMockRunner.class)
@PrepareForTest({Util.class})//声明要Mock的类
public class UtilTest {
    @Test
    public void staicMethodMockTest() throws Exception {
        PowerMock.mockStatic(System.class);//Mock静态方法
        EasyMock.expect(System.getenv("java_home"))).andReturn("env value");//录制Mock对象的静态方法
        PowerMock.replayAll();//重放Mock对象
        Assert.assertEquals("env value",Util.getEnv("java_home"));
        PowerMock.verifyAll();//验证Mock对象
    }
}
```
静态的final方法也是如此。这里需要注意的是：对于JDK的类如果要进行静态或final方法Mock时，@PrepareForTest()注解中只能放被测试的类，而非JDK的类，如上面例子中的UtilTest.class。对于非JDK的类如果需要进行静态final方法Mock时， @PrepareForTest()注解中直接放方法所在的类，如果上面例子中的System不是JDK的类，则可以直接放System.class。@PrepareForTest({......}) 注解既可以加在类层次上（对整个测试文件有效），也可以加在测试方法上（只对测试方法有效）。

+ 非静态的final方法

和非静态的一样。对于非静态的，如下：
```
public class FinalClass {
    public final String finalMethod() {
       return "final method";
    }
}

public class FinalUtil{
    public String  callFinalMethod(FinalClass cls) {
        return cls.getName;
    }
}
   
@RunWith(PowerMockRunner.class)
public class FinalMethodMockTest {
  @Test
  @PrepareForTest(FinalClass.class)
  public void testCallFinalMethod() {
      FinalClass cls = PowerMock.createMock(FinalClass.class); //创建Mock对象
      FinalUtil util = new FinalUtil();
      EasyMock.expect(cls.finalMethod()).andReturn("cls");
      PowerMock.replayAll();                                      
      Assert.assertTrue(util.callFinalMethod(cls));
      PowerMock.verifyAll();
  }
}
```
+ 私有方法
```
public class PrivateMockClass
    private boolean privateMethod(final String name) {
        return true;
    }
}
   
@RunWith(PowerMockRunner.class)
@PrepareForTest(PrivateMockClass.class)
public class PrivatMethodMockTest {
  @Test
    public void testPrivateMock() throws Exception {
        PrivateMockClass mock = PowerMock.createPartialMock(PrivateMockClass.class, “privateMethod”);//只对privateMethod方法Mock
        PowerMock.expectPrivate(mock, "privateMethod", "name").andReturn(true);//录制
        PowerMock.replay(mock);
        assertTrue(mock.privateMethod(“name”));
        PowerMock.verify(mock);
    }
}
```
此种方式除了私有方法，还可以用在被测试方法都在同一个类，且不容易创建的场景下。

除了上述场景之外，经常会遇到在测试方法中调用构造函数产生对象。那么如何对构造方法进行mock呢？
```
File mock = createMockAndExpectNew(File.class,“mockPath”);
EasyMock.expect(mock.exists()).andReturn(false);
EasyMock.expect(mock.mkdirs()).andReturn(true);
PowerMock.replay(mock, File.class);

...

PowerMock.verify(mock, File.class);
```
以上代码即可完成对File的构造方法的mock（对于mockPath的File，其exists和mkdirs方法都返回true）。

# 注意
由上述可知，对于单元测试，MOCK是一件比较麻烦的事情。因此，对依赖于外部环境（第三方服务调用、数据库访问）的代码做单元测试非常不方便。这造成了很多开发团队没有自动化单元测试，或者花费了不小的功夫才实现了自动化单元测试。其实对诸如调用业务逻辑、读写存储设备、粘合业务逻辑和存储操作等的代码是无需单元测试的。这些代码的核心在于传递上下文的数据，会依赖于外部环境，只需要做通过集成测试来测试其联通性即可。单元测试应该针对的是开发人员自己的业务代码。因此，能够单元测试、应该进行单元测试的代码应该是不依赖于外部环境的业务代码。判断要测试的方法是否是可单元测试代码可以通过以下两种方式：

+ 该方法能否用一个main方法就能直接运行。
+ 该方法的参数能否不依赖外部环境而进行自由模拟。
对于一些将访问代码糅杂在业务逻辑里无法单元测试的方法，可以通过将需要的数据从上下文中抽离出来达到不依赖外部环境的目的，如：
```
@Resource
private UserDao userDao;

public void methodA（）{
	User u = userDao.getById(1);
	….//业务逻辑
}
```
可以重构为
```
public void methodA（User u）{
	….//业务逻辑
}
```
这样，可以自由模拟User数据，来对此方法做单元测试。

此外，编写单元测试还需要注意：

保持单元测试用例的运行速度，不要将性能和大的集成用例放在单元测试中。
保持单元测试的每个用例都用 try...finally 或 tearDown 释放资源。