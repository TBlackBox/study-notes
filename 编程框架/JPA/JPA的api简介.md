# JPA的api简单介绍
##  Persistence对象
Persistence对象主要作用是用于获取EntityManagerFactory对象的 。通过调用该类的createEntityManagerFactory静态方法，根据配置文件中持久化单元名称创建EntityManagerFactory。
```
/**
* 创建实体管理类工厂，借助Persistence的静态方法获取
*其中传递的参数为持久化单元名称(herbernate配置文件里面的"persistence-unit"的名字)，需要*jpa配置文件中指定
*/
EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
```

## EntityManagerFactory
EntityManagerFactory 是一个接口

### 作用
主要用来创建 EntityManager 实例。（不同的JPA的实现方式，需要实现这个接口，用户生产Entitymanager）

### 线程安全的
由于EntityManagerFactory 是一个线程安全的对象（即多个线程访问同一个EntityManagerFactory 对象不会有线程安全问题），并且EntityManagerFactory 的创建极其浪费资源，所以在使用JPA编程时，我们可以对EntityManagerFactory 的创建进行优化，只需要做到一个工程只存在一个EntityManagerFactory 即可

````
//创建实体管理类
EntityManager em = factory.createEntityManager();
```

## EntityManager
EntityManager 接口，在 JPA 规范中, EntityManager是完成持久化操作的核心对象。实体类作为普通 java对象，只有在调用 EntityManager将其持久化后才会变成持久化对象。
EntityManager对象在一组实体类与底层数据源之间进行 O/R 映射的管理。它可以用来管理和更新 Entity Bean, 根椐主键查找 Entity Bean, 还可以通过JPQL语句查询实体。
### EntityManager的一些方法

```
    public void persist(Object entity);
    public <T> T merge(T entity); 
    public void remove(Object entity);
    public <T> T find(Class<T> entityClass, Object primaryKey);
    public <T> T find(Class<T> entityClass, Object primaryKey, 
                      Map<String, Object> properties); 
    public <T> T find(Class<T> entityClass, Object primaryKey,
                      LockModeType lockMode);
    public <T> T find(Class<T> entityClass, Object primaryKey,
                      LockModeType lockMode, 
                      Map<String, Object> properties);
    public <T> T getReference(Class<T> entityClass, 
                                  Object primaryKey);
    public void setFlushMode(FlushModeType flushMode);
    public FlushModeType getFlushMode();
    public void lock(Object entity, LockModeType lockMode);
    public void lock(Object entity, LockModeType lockMode,
                     Map<String, Object> properties);    
    public void refresh(Object entity);
    public void refresh(Object entity,
                            Map<String, Object> properties); 
    public void refresh(Object entity, LockModeType lockMode);

    public void refresh(Object entity, LockModeType lockMode,
                        Map<String, Object> properties);
    public void clear();
    public void detach(Object entity); 
    public boolean contains(Object entity);
    public LockModeType getLockMode(Object entity);
    public void setProperty(String propertyName, Object value);
    public Map<String, Object> getProperties();
    public Query createQuery(String qlString);
    public <T> TypedQuery<T> createQuery(CriteriaQuery<T> criteriaQuery); 
    public Query createQuery(CriteriaUpdate updateQuery);
    public Query createQuery(CriteriaDelete deleteQuery);
    public <T> TypedQuery<T> createQuery(String qlString, Class<T> resultClass);
    public Query createNamedQuery(String name);
    public <T> TypedQuery<T> createNamedQuery(String name, Class<T> resultClass);
    public Query createNativeQuery(String sqlString);
    public Query createNativeQuery(String sqlString, Class resultClass);
    public Query createNativeQuery(String sqlString, String resultSetMapping);
    public StoredProcedureQuery createNamedStoredProcedureQuery(String name);
    public StoredProcedureQuery createStoredProcedureQuery(String procedureName);
    public StoredProcedureQuery createStoredProcedureQuery(
	       String procedureName, Class... resultClasses);
    public StoredProcedureQuery createStoredProcedureQuery(
              String procedureName, String... resultSetMappings);
    public void joinTransaction();
    public boolean isJoinedToTransaction();
    public <T> T unwrap(Class<T> cls); 
    public Object getDelegate();
    public void close();
    public boolean isOpen();
    public EntityTransaction getTransaction();
    public EntityManagerFactory getEntityManagerFactory();
    public CriteriaBuilder getCriteriaBuilder();
    public Metamodel getMetamodel();
    public <T> EntityGraph<T> createEntityGraph(Class<T> rootType);
    public EntityGraph<?> createEntityGraph(String graphName);
    public  EntityGraph<?> getEntityGraph(String graphName);
    public <T> List<EntityGraph<? super T>> getEntityGraphs(Class<T> entityClass);

```


### 通过调用EntityManager的方法完成获取事务，以及持久化数据库的操作
```
getTransaction : 获取事务对象
persist ： 保存操作
merge ： 更新操作
remove ： 删除操作
find/getReference ： 根据id查询 find(立即查询)  getReference（懒加载，即需要的时候才会去查询）
```

## EntityTransaction
EntityTransaction接口，在 JPA 规范中, EntityTransaction是完成事务操作的核心对象，对于EntityTransaction在我们的java代码中承接的功能比较简单

### 几个功能
```
//开启事务
public void begin();
//提交事务
public void commit();
//回滚事务
public void rollback();
public void setRollbackOnly();
public boolean getRollbackOnly();
public boolean isActive();
```

# 例子
```
/**
* 创建实体管理类工厂，借助Persistence的静态方法获取
* 		其中传递的参数为持久化单元名称，需要jpa配置文件中指定
*/
EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
//创建实体管理类
EntityManager em = factory.createEntityManager();
//获取事务对象
EntityTransaction tx = em.getTransaction();
//开启事务
tx.begin();
Customer c = new Customer();
c.setCustName("哈哈");
//保存操作
em.persist(c);
//提交事务
tx.commit();
//释放资源
em.close();
factory.close();

```