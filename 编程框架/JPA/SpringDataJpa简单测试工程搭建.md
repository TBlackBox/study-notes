# 目标
Spring Data JPA完成客户的基本CRUD操作

# 搭建方式

## 创建一个maven工程
创建一个maven工程，并引出下面的坐标
*** 使用Spring Data JPA，需要整合Spring与Spring Data JPA，并且需要提供JPA的服务提供者hibernate，所以需要导入spring相关坐标，hibernate坐标，数据库驱动坐标等 ***
```
  <properties>
        <spring.version>5.0.2.RELEASE</spring.version>
        <hibernate.version>5.0.7.Final</hibernate.version>
        <slf4j.version>1.6.6</slf4j.version>
        <log4j.version>1.2.12</log4j.version>
        <c3p0.version>0.9.1.2</c3p0.version>
        <mysql.version>5.1.6</mysql.version>
    </properties>

    <dependencies>
        <!-- junit单元测试 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <!-- spring beg -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.6.8</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!-- spring对orm框架的支持包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!-- spring end -->

        <!-- hibernate beg -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.2.1.Final</version>
        </dependency>
        <!-- hibernate end -->

        <!-- c3p0 beg -->
        <dependency>
            <groupId>c3p0</groupId>
            <artifactId>c3p0</artifactId>
            <version>${c3p0.version}</version>
        </dependency>
        <!-- c3p0 end -->

        <!-- log end -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <!-- log end -->


        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>

        <!-- spring data jpa 的坐标-->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
            <version>1.9.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!-- el beg 使用spring data jpa 必须引入 -->
        <dependency>
            <groupId>javax.el</groupId>
            <artifactId>javax.el-api</artifactId>
            <version>2.2.4</version>
        </dependency>

        <dependency>
            <groupId>org.glassfish.web</groupId>
            <artifactId>javax.el</artifactId>
            <version>2.2.4</version>
        </dependency>
        <!-- el end -->
    </dependencies>
```

## 整合Spring Data JPA与Spring
在classapath下（即放在resources下）创建`applicationContext.xml`文件，并把Spring Data HPA注入进去
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa" xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
		http://www.springframework.org/schema/data/jpa
		http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <!--spring 和 spring data jpa的配置-->

    <!-- 1.创建entityManagerFactory对象交给spring容器管理-->
    <bean id="entityManagerFactoty" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <!--配置的扫描的包（实体类所在的包） -->
        <property name="packagesToScan" value="cn.itcast.domain" />
        <!-- jpa的实现厂家 -->
        <property name="persistenceProvider">
            <bean class="org.hibernate.jpa.HibernatePersistenceProvider"/>
        </property>

        <!--jpa的供应商适配器 -->
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <!--配置是否自动创建数据库表 -->
                <property name="generateDdl" value="false" />
                <!--指定数据库类型 -->
                <property name="database" value="MYSQL" />
                <!--数据库方言：支持的特有语法 -->
                <property name="databasePlatform" value="org.hibernate.dialect.MySQLDialect" />
                <!--是否显示sql -->
                <property name="showSql" value="true" />
            </bean>
        </property>

        <!--jpa的方言 ：高级的特性   （可以理解为一个技术的一些高级特性，特有的语法）-->
        <property name="jpaDialect" >
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect" />
        </property>

    </bean>

    <!--2.创建数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="user" value="root"></property>
        <property name="password" value="111111"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///jpa" ></property>
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    </bean>

    <!--3.整合spring dataJpa-->
     <!--base-package  spring data jpa 的接口都放在下面-->
    <jpa:repositories base-package="cn.itcast.dao" transaction-manager-ref="transactionManager"
                   entity-manager-factory-ref="entityManagerFactoty" ></jpa:repositories>

    <!--4.配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactoty"></property>
    </bean>

	<!-- 配置声明式事务  可以不用的-->
    <!-- 4.txAdvice-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!-- 5.aop-->
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* cn.itcast.service.*.*(..))" />
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut" />
    </aop:config>


    <!--5.声明式事务 -->

    <!-- 6. 配置包扫描-->
    <context:component-scan base-package="cn.itcast" ></context:component-scan>
</beans>
```

## 配置实体和数据库的映射关系
```
import javax.persistence.*;

/**
 * 1.实体类和表的映射关系
 *      @Eitity
 *      @Table
 * 2.类中属性和表中字段的映射关系
 *      @Id
 *      @GeneratedValue
 *      @Column
 */
@Entity
@Table(name="cst_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="cust_id")
    private Long custId;
    @Column(name="cust_address")
    private String custAddress;
    @Column(name="cust_industry")
    private String custIndustry;
    @Column(name="cust_level")
    private String custLevel;
    @Column(name="cust_name")
    private String custName;
    @Column(name="cust_phone")
    private String custPhone;
    @Column(name="cust_source")
    private String custSource;

   //这里省略了get set toString 方法
}

```
## 编写符合Spring Data JPA规范的Dao层接口
Spring Data JPA是spring提供的一款对于数据访问层（Dao层）的框架，使用Spring Data JPA，只需要按照框架的规范提供dao接口，不需要实现类就可以完成数据库的增删改查、分页查询等方法的定义，极大的简化了我们的开发过程。

***在Spring Data JPA中，对于定义符合规范的Dao层接口，我们只需要遵循以下几点就可以了：***

1. 创建一个Dao层接口，并实现JpaRepository和JpaSpecificationExecutor
2. 提供相应的泛型

*** 特别说明：基本的需求不需要定义接口里面的内容，里面都内容主要书将的其他几种操作方式的一个说明***

```
import cn.itcast.domain.Customer;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;

import java.util.List;

/**
 * 符合SpringDataJpa的dao层接口规范
 *      JpaRepository<操作的实体类类型，实体类中主键属性的类型>
 *          * 封装了基本CRUD操作
 *      JpaSpecificationExecutor<操作的实体类类型>
 *          * 封装了复杂查询（分页）
 */
public interface CustomerDao extends JpaRepository<Customer,Long> ,JpaSpecificationExecutor<Customer> {

    /**
     * 案例：根据客户名称查询客户
     *      使用jpql的形式查询
     *  jpql：from Customer where custName = ?
     *
     *  配置jpql语句，使用的@Query注解
     */
    @Query(value="from Customer where custName = ?")
    public Customer findJpql(String custName);


    /**
     * 案例：根据客户名称和客户id查询客户
     *      jpql： from Customer where custName = ? and custId = ?
     *
     *  对于多个占位符参数
     *      赋值的时候，默认的情况下，占位符的位置需要和方法参数中的位置保持一致
     *
     *  可以指定占位符参数的位置
     *      ? 索引的方式，指定此占位的取值来源
     */
    @Query(value = "from Customer where custName = ?2 and custId = ?1")
    public Customer findCustNameAndId(Long id,String name);

    /**
     * 使用jpql完成更新操作
     *      案例 ： 根据id更新，客户的名称
     *          更新4号客户的名称，将名称改为“黑马程序员”
     *
     *  sql  ：update cst_customer set cust_name = ? where cust_id = ?
     *  jpql : update Customer set custName = ? where custId = ?
     *
     *  @Query : 代表的是进行查询
     *      * 声明此方法是用来进行更新操作
     *  @Modifying
     *      * 当前执行的是一个更新操作
     *
     */
    @Query(value = " update Customer set custName = ?2 where custId = ?1 ")
    @Modifying
    public void updateCustomer(long custId,String custName);


    /**
     * 使用sql的形式查询：
     *     查询全部的客户
     *  sql ： select * from cst_customer;
     *  Query : 配置sql查询
     *      value ： sql语句
     *      nativeQuery ： 查询方式
     *          true ： sql查询
     *          false：jpql查询
     *
     */
    //@Query(value = " select * from cst_customer" ,nativeQuery = true)
    @Query(value="select * from cst_customer where cust_name like ?1",nativeQuery = true)
    public List<Object [] > findSql(String name);


    /**
     * 方法名的约定：
     *      findBy : 查询
     *            对象中的属性名（首字母大写） ： 查询的条件
     *            CustName
     *            * 默认情况 ： 使用 等于的方式查询
     *                  特殊的查询方式
     *
     *  findByCustName   --   根据客户名称查询
     *
     *  再springdataJpa的运行阶段
     *          会根据方法名称进行解析  findBy    from  xxx(实体类)
     *                                      属性名称      where  custName =
     *
     *      1.findBy  + 属性名称 （根据属性名称进行完成匹配的查询=）
     *      2.findBy  + 属性名称 + “查询方式（Like | isnull）”
     *          findByCustNameLike
     *      3.多条件查询
     *          findBy + 属性名 + “查询方式”   + “多条件的连接符（and|or）”  + 属性名 + “查询方式”
     */
    public Customer findByCustName(String custName);


    public List<Customer> findByCustNameLike(String custName);

    //使用客户名称模糊匹配和客户所属行业精准匹配的查询
    public Customer findByCustNameLikeAndCustIndustry(String custName,String custIndustry);
}

```

## 完成一般的CURD
```
import cn.itcast.dao.CustomerDao;
import cn.itcast.domain.Customer;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class) //声明spring提供的单元测试环境
@ContextConfiguration(locations = "classpath:applicationContext.xml")//指定spring容器的配置信息
public class CustomerDaoTest {
    @Autowired
    private CustomerDao customerDao;

    /**
     * 根据id查询
     */
    @Test
    public void testFindOne() {
        Customer customer = customerDao.findOne(4l);
        System.out.println(customer);
    }

    /**
     * save : 保存或者更新
     *      根据传递的对象是否存在主键id，
     *      如果没有id主键属性：保存
     *      存在id主键属性，根据id查询数据，更新数据
     */
    @Test
    public void testSave() {
        Customer customer  = new Customer();
        customer.setCustName("黑马程序员");
        customer.setCustLevel("vip");
        customer.setCustIndustry("it教育");
        customerDao.save(customer);
    }

    @Test
    public void testUpdate() {
        Customer customer  = new Customer();
        customer.setCustId(4l);
        customer.setCustName("黑马程序员很厉害");
        customerDao.save(customer);
    }

    @Test
    public void testDelete () {
        customerDao.delete(3l);
    }


    /**
     * 查询所有
     */
    @Test
    public void testFindAll() {
        List<Customer> list = customerDao.findAll();
        for(Customer customer : list) {
            System.out.println(customer);
        }
    }

    /**
     * 测试统计查询：查询客户的总数量
     *      count:统计总条数
     */
    @Test
    public void testCount() {
        long count = customerDao.count();//查询全部的客户数量
        System.out.println(count);
    }

    /**
     * 测试：判断id为4的客户是否存在
     *      1. 可以查询以下id为4的客户
     *          如果值为空，代表不存在，如果不为空，代表存在
     *      2. 判断数据库中id为4的客户的数量
     *          如果数量为0，代表不存在，如果大于0，代表存在
     */
    @Test
    public void  testExists() {
        boolean exists = customerDao.exists(4l);
        System.out.println("id为4的客户 是否存在："+exists);
    }


    /**
     * 根据id从数据库查询
     *      @Transactional : 保证getOne正常运行
     *
     *  findOne：
     *      em.find()           :立即加载
     *  getOne：
     *      em.getReference     :延迟加载
     *      * 返回的是一个客户的动态代理对象
     *      * 什么时候用，什么时候查询
     */
    @Test
    @Transactional
    public void  testGetOne() {
        Customer customer = customerDao.getOne(4l);
        System.out.println(customer);
    }

}

```

## 原理简单解析

在使用Spring Data JPA时，一般实现JpaRepository和JpaSpecificationExecutor接口，这样就可以使用这些接口中定义的方法，但是这些方法都只是一些声明，没有具体的实现方式，那么在 Spring Data JPA中它又是怎么实现的呢？
***通过dubug看赛 ，分析赛***
当程序执行的时候，会通过JdkDynamicAopProxy的invoke方法，对customerDao对象生成动态代理对象。根据对Spring Data JPA介绍而知，要想进行findOne查询方法，最终还是会出现JPA规范的API完成操作，那么这些底层代码存在于何处呢？答案很简单，都隐藏在通过JdkDynamicAopProxy生成的动态代理对象当中，而这个动态代理对象就是SimpleJpaRepository。

原理剖析
1. 通过JdkDynamicAopProxy的invoke方法创建了一个动态代理对象
2. SimpleJpaRepository当中封装了JPA的操作（借助JPA的api完成数据库的CRUD）
3. 通过hibernate完成数据库操作（封装了jdbc）

## 几种查询方式
1. 在继承JpaRepository，和JpaRepository接口后,我们就可以使用接口中定义的方法进行查询
```
findOne（id） ：根据id查询
save(customer):保存或者更新（依据：传递的实体类对象中，是否包含id属性）
delete（id） ：根据id删除
findAll() : 查询全部
```
还有介绍继承JpaSpecificationExecutor的方法列表
***注：也就是上面的测试例子***
2. 使用JPQL的方式查询
使用Spring Data JPA提供的查询方法已经可以解决大部分的应用场景，但是对于某些业务来说，我们还需要灵活的构造查询条件，这时就可以使用@Query注解，结合JPQL的语句方式完成查询

***@Query 注解的使用非常简单，只需在方法上面标注该注解，同时提供一个JPQL查询语句即可***

3. 使用SQL语句查询
Spring Data JPA同样也支持sql语句的查询

4. 方法命名规则查询
顾名思义，方法命名规则查询就是根据方法的名字，就能创建查询。只需要按照Spring Data JPA提供的方法命名规则定义方法的名称，就可以完成查询工作。Spring Data JPA在程序执行的时候会根据方法名称进行解析，并自动生成查询语句进行查询

按照Spring Data JPA 定义的规则，查询方法以findBy开头，涉及条件查询时，条件的属性用条件关键字连接，要注意的是：条件属性首字母需大写。框架在进行方法名解析时，会先把方法名多余的前缀截取掉，然后对剩下部分进行解析。

具体的关键字，使用方法和生产成SQL如下表所示


|Keyword	|Sample	|JPQL|
|-------------|----------|----------|
|And	|findByLastnameAndFirstname|	… where x.lastname = ?1 and x.firstname = ?2|
|Or|	findByLastnameOrFirstname|	… where x.lastname = ?1 or x.firstname = ?2		|
|Is,Equals	|findByFirstnameIs,findByFirstnameEquals|	… where x.firstname = ?1		|
|Between|	findByStartDateBetween|	… where x.startDate between ?1 and ?2	|
|LessThan|	findByAgeLessThan|	… where x.age < ?1		|
|LessThanEqual|	findByAgeLessThanEqual|	… where x.age ⇐ ?1		|
|GreaterThan|	findByAgeGreaterThan|	… where x.age > ?1		|
|GreaterThanEqual|	findByAgeGreaterThanEqual|	… where x.age >= ?1		|
|After|	findByStartDateAfter|	… where x.startDate > ?1		|
|Before|	findByStartDateBefore|	… where x.startDate < ?1		|
|IsNull|	findByAgeIsNull|	… where x.age is null		|
|IsNotNull,NotNull|	findByAge(Is)NotNull|	… where x.age not null	|
|Like|	findByFirstnameLike|	… where x.firstname like ?1		|
|NotLike|	findByFirstnameNotLike|	… where x.firstname not like ?1		|
|StartingWith|	findByFirstnameStartingWith|	… where x.firstname like ?1 (parameter bound with appended %)	|
|EndingWith|	findByFirstnameEndingWith|	… where x.firstname like ?1 (parameter bound with prepended %)		|
|Containing|	findByFirstnameContaining|	… where x.firstname like ?1 (parameter bound wrapped in %)		|
|OrderBy|	findByAgeOrderByLastnameDesc|	… where x.age = ?1 order by x.lastname desc		|
|Not|	findByLastnameNot|	… where x.lastname <> ?1		|
|In|	findByAgeIn(Collection ages)|	… where x.age in ?1		|
|NotIn|	findByAgeNotIn(Collection age)|	… where x.age not in ?1	|
|TRUE|	findByActiveTrue()|	… where x.active = true		|
|FALSE|	findByActiveFalse()|	… where x.active = false		|
|IgnoreCase|	findByFirstnameIgnoreCase|	… where UPPER(x.firstame) = UPPER(?1)	|


后面3种查询方式可以看dao文件的例子

# 其他查询方式的调用例子
```java
import cn.itcast.dao.CustomerDao;
import cn.itcast.domain.Customer;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.jpa.repository.Query;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import java.util.Arrays;
import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class) //声明spring提供的单元测试环境
@ContextConfiguration(locations = "classpath:applicationContext.xml")//指定spring容器的配置信息
public class JpqlTest {
    @Autowired
    private CustomerDao customerDao;

    @Test
    public void  testFindJPQL() {
        Customer customer = customerDao.findJpql("传智播客");
        System.out.println(customer);
    }


    @Test
    public void testFindCustNameAndId() {
       // Customer customer =  customerDao.findCustNameAndId("传智播客",1l);
        Customer customer =  customerDao.findCustNameAndId(1l,"传智播客");
        System.out.println(customer);
    }

    /**
     * 测试jpql的更新操作
     *  * springDataJpa中使用jpql完成 更新/删除操作
     *         * 需要手动添加事务的支持
     *         * 默认会执行结束之后，回滚事务
     *   @Rollback : 设置是否自动回滚
     *          false | true
     */
    @Test
    @Transactional //添加事务的支持
    @Rollback(value = false)
    public void testUpdateCustomer() {
        customerDao.updateCustomer(4l,"黑马程序员");
    }

    //测试sql查询
    @Test
    public void testFindSql() {
        List<Object[]> list = customerDao.findSql("传智播客%");
        for(Object [] obj : list) {
            System.out.println(Arrays.toString(obj));
        }
    }


    //测试方法命名规则的查询
    @Test
    public void testNaming() {
        Customer customer = customerDao.findByCustName("传智播客");
        System.out.println(customer);
    }


    //测试方法命名规则的查询
    @Test
    public void testFindByCustNameLike() {
        List<Customer> list = customerDao.findByCustNameLike("传智播客%");
        for (Customer customer : list) {
            System.out.println(customer);
        }
    }


    //测试方法命名规则的查询
    @Test
    public void testFindByCustNameLikeAndCustIndustry() {
        Customer customer = customerDao.findByCustNameLikeAndCustIndustry("传智播客1%", "it教育");
        System.out.println(customer);
    }
}

```
# 动态查询

## 说明
有时我们在查询某个实体的时候，给定的条件是不固定的，这时就需要动态构建相应的查询语句，在Spring Data JPA中可以通过JpaSpecificationExecutor接口查询。相比JPQL,其优势是类型安全,更加的面向对象。
```
import java.util.List;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;

/**
 *	JpaSpecificationExecutor中定义的方法
 **/
 public interface JpaSpecificationExecutor<T> {
   	//根据条件查询一个对象
 	T findOne(Specification<T> spec);	
   	//根据条件查询集合
 	List<T> findAll(Specification<T> spec);
   	//根据条件分页查询
 	Page<T> findAll(Specification<T> spec, Pageable pageable);
   	//排序查询查询
 	List<T> findAll(Specification<T> spec, Sort sort);
   	//统计查询
 	long count(Specification<T> spec);
}
```
对于JpaSpecificationExecutor，这个接口基本是围绕着Specification接口来定义的。我们可以简单的理解为，Specification构造的就是查询条件。

|方法名称|	Sql对应关系|
|--------|---------|
|equle|	filed = value|
|gt（greaterThan ）|	filed > value|
|lt（lessThan ）|	filed < value|
|ge（greaterThanOrEqualTo ）|	filed >= value|
|le（ lessThanOrEqualTo）|	filed <= value|
|notEqule|	filed != value|
|like|	filed like value|
|notLike|	filed not like value|

## 使用方式
```
import cn.itcast.dao.CustomerDao;
import cn.itcast.domain.Customer;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.persistence.criteria.*;
import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class SpecTest {

    @Autowired
    private CustomerDao customerDao;

    /**
     * 根据条件，查询单个对象
     *
     */
    @Test
    public void testSpec() {
        //匿名内部类
        /**
         * 自定义查询条件
         *      1.实现Specification接口（提供泛型：查询的对象类型）
         *      2.实现toPredicate方法（构造查询条件）
         *      3.需要借助方法参数中的两个参数（
         *          root：获取需要查询的对象属性
         *          CriteriaBuilder：构造查询条件的，内部封装了很多的查询条件（模糊匹配，精准匹配）
         *       ）
         *  案例：根据客户名称查询，查询客户名为传智播客的客户
         *          查询条件
         *              1.查询方式
         *                  cb对象
         *              2.比较的属性名称
         *                  root对象
         *
         */
        Specification<Customer> spec = new Specification<Customer>() {
            @Override
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                //1.获取比较的属性
                Path<Object> custName = root.get("custId");
                //2.构造查询条件  ：    select * from cst_customer where cust_name = '传智播客'
                /**
                 * 第一个参数：需要比较的属性（path对象）
                 * 第二个参数：当前需要比较的取值
                 */
                Predicate predicate = cb.equal(custName, "传智播客");//进行精准的匹配  （比较的属性，比较的属性的取值）
                return predicate;
            }
        };
        Customer customer = customerDao.findOne(spec);
        System.out.println(customer);
    }

    /**
     * 多条件查询
     *      案例：根据客户名（传智播客）和客户所属行业查询（it教育）
     *
     */
    @Test
    public void testSpec1() {
        /**
         *  root:获取属性
         *      客户名
         *      所属行业
         *  cb：构造查询
         *      1.构造客户名的精准匹配查询
         *      2.构造所属行业的精准匹配查询
         *      3.将以上两个查询联系起来
         */
        Specification<Customer> spec = new Specification<Customer>() {
            @Override
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                Path<Object> custName = root.get("custName");//客户名
                Path<Object> custIndustry = root.get("custIndustry");//所属行业

                //构造查询
                //1.构造客户名的精准匹配查询
                Predicate p1 = cb.equal(custName, "传智播客");//第一个参数，path（属性），第二个参数，属性的取值
                //2..构造所属行业的精准匹配查询
                Predicate p2 = cb.equal(custIndustry, "it教育");
                //3.将多个查询条件组合到一起：组合（满足条件一并且满足条件二：与关系，满足条件一或满足条件二即可：或关系）
                Predicate and = cb.and(p1, p2);//以与的形式拼接多个查询条件
                // cb.or();//以或的形式拼接多个查询条件
                return and;
            }
        };
        Customer customer = customerDao.findOne(spec);
        System.out.println(customer);
    }


    /**
     * 案例：完成根据客户名称的模糊匹配，返回客户列表
     *      客户名称以 ’传智播客‘ 开头
     *
     * equal ：直接的到path对象（属性），然后进行比较即可
     * gt，lt,ge,le,like : 得到path对象，根据path指定比较的参数类型，再去进行比较
     *      指定参数类型：path.as(类型的字节码对象)
     */
    @Test
    public void testSpec3() {
        //构造查询条件
        Specification<Customer> spec = new Specification<Customer>() {
            @Override
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                //查询属性：客户名
                Path<Object> custName = root.get("custName");
                //查询方式：模糊匹配
                Predicate like = cb.like(custName.as(String.class), "传智播客%");
                return like;
            }
        };
//        List<Customer> list = customerDao.findAll(spec);
//        for (Customer customer : list) {
//            System.out.println(customer);
//        }
        //添加排序
        //创建排序对象,需要调用构造方法实例化sort对象
        //第一个参数：排序的顺序（倒序，正序）
        //   Sort.Direction.DESC:倒序
        //   Sort.Direction.ASC ： 升序
        //第二个参数：排序的属性名称
        Sort sort = new Sort(Sort.Direction.DESC,"custId");
        List<Customer> list = customerDao.findAll(spec, sort);
        for (Customer customer : list) {
            System.out.println(customer);
        }
    }

    /**
     * 分页查询
     *      Specification: 查询条件
     *      Pageable：分页参数
     *          分页参数：查询的页码，每页查询的条数
     *          findAll(Specification,Pageable)：带有条件的分页
     *          findAll(Pageable)：没有条件的分页
     *  返回：Page（springDataJpa为我们封装好的pageBean对象，数据列表，共条数）
     */
    @Test
    public void testSpec4() {

        Specification spec = null;
        //PageRequest对象是Pageable接口的实现类
        /**
         * 创建PageRequest的过程中，需要调用他的构造方法传入两个参数
         *      第一个参数：当前查询的页数（从0开始）
         *      第二个参数：每页查询的数量
         */
        Pageable pageable = new PageRequest(0,2);
        //分页查询
        Page<Customer> page = customerDao.findAll(null, pageable);
        System.out.println(page.getContent()); //得到数据集合列表
        System.out.println(page.getTotalElements());//得到总条数
        System.out.println(page.getTotalPages());//得到总页数
    }
}

```