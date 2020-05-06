# Repository
- 提供了方法名成查询方式: 
　方法的名称要遵循 findBy + 属性名（首字母大写） + 查询条件(首字母大写 Is Equals)
```
findByNameLike(String name)
findByName(String name)
findByNameAndAge(String name, Integer age)
findByNameOrAddress(String name) 等...
```
- 基于@Query注解的查询和更新
```
/**
* SQL nativeQuery的值是true 执行的时候不用再转化,也就是本地sql 查询
* @param name
* @return
*/
@Query(value = "SELECT * FROM table_user WHERE name = ?1", nativeQuery = true)
List<User> findByUsernameSQL(String name);
```
- 基于HQL
```
@Query("Update User set name = ?1 WHERE id = ?2")
@Modifying
int updateNameAndId(String name, Integer id);
```

注解说明：  
1. @Query   
代表的是进行查询  
value ： sql语句   
nativeQuery ： 查询方式  
     - true ： sql查询  
     - false：jpql查询  
2. @Modifying
声明此方法是用来进行更新操作，当前执行的是一个更新操作


# CrudReposiroty 

继承了Repository,主要是添加了对数据的增删改查的方法。
```
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
            <S extends T> S save(S entity);
            <S extends T> Iterable<S> saveAll(Iterable<S> entities);
            Optional<T> findById(ID id);
            boolean existsById(ID id);
            Iterable<T> findAll();
            Iterable<T> findAllById(Iterable<ID> ids);
            long count();
            void deleteById(ID id);
            void delete(T entity);
            void deleteAll(Iterable<? extends T> entities);
            void deleteAll();
}
```

# PagingAndSortingRepository
继承了CrudRepository,主要是排序和分页

```
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
	Iterable<T> findAll(Sort sort);
	Page<T> findAll(Pageable pageable);
}
```

# JPARepository
继承了`PagingAndSortingRepository`接口和`QueryByExampleExecutor`接口
## 优点
对继承父接口中方法的返回值进行了适配,因为在父类接口中通常都返回迭代器，需要我们自己进行强制类型转化。而在JpaRepository中，直接返回了List

```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.data.jpa.repository;

import java.util.List;
import org.springframework.data.domain.Example;
import org.springframework.data.domain.Sort;
import org.springframework.data.repository.NoRepositoryBean;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.repository.query.QueryByExampleExecutor;

@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    List<T> findAll();

    List<T> findAll(Sort var1);

    List<T> findAllById(Iterable<ID> var1);

    <S extends T> List<S> saveAll(Iterable<S> var1);

    void flush();

    <S extends T> S saveAndFlush(S var1);

    void deleteInBatch(Iterable<T> var1);

    void deleteAllInBatch();

    T getOne(ID var1);

    <S extends T> List<S> findAll(Example<S> var1);

    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}

JpaRepository
```

# JpaSpecificationExecutor
这个接口单独存在，没有继承以上说的接口
## 作用
主要提供了多条件查询的支持，并且可以在查询中添加分页和排序，动态查询等功能。
```
public interface JpaSpecificationExecutor<T> {
	Optional<T> findOne(@Nullable Specification<T> spec);
	List<T> findAll(@Nullable Specification<T> spec);
	Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);
	List<T> findAll(@Nullable Specification<T> spec, Sort sort);
	long count(@Nullable Specification<T> spec);
}
```

# 注意`@NoRepositoryBean`
Spring Data JPA @NoRepositoryBean接口   
       如果您正在使用Spring命名空间使用Spring命名空间进行自动存储库接口检测，那将导致Spring尝试创建MyRepository实例。这当然不是所希望的，因为它只是在Repository和您要为每个实体定义的实际存储库接口之间起中间作用。要将扩展存储库的接口排除在实例化为存储库实例之外，请使用@NoRepositoryBean。

自己通俗的讲连接：
```
相当于你在使用spring data jpa 的时候，每个实体类有需要实现的相同的方法，就可以单独抽取出来，放在一个公共的接口MyRepository中，并这个类继承了jpa的相关Repository接口或类，由MyRepository接口来衔接jpa的相关操作，其他实体类需要实现的操作就直接继承MyRepository接口，不用每次都去继承jpa的相关接口或类啦，所以这个公共接口就需要这个注解@NoRepositoryBean来标识.
也可以这样理解：
SpringDatajap会为继承的的`Repository`进行存储库代理bean，加上他就不会了
```
 

另一解释：

注释用于避免为实际匹配repo接口条件但不打算成为一个接口的接口创建存储库代理。只有在您开始使用功能扩展所有存储库时才需要它。



# 总结
Spring Data Jpa中一共提供了
```                      
Repository：  
　　提供了findBy + 属性方法   
　　@Query   
　　        HQL： nativeQuery 默认false  
　　        SQL: nativeQuery 默认true  
更新的时候，需要配合@Modifying使用
            CurdRepository:
                        继承了Repository 主要提供了对数据的增删改查
            PagingAndSortRepository:
                        继承了CrudRepository 提供了对数据的分页和排序，缺点是只能对所有的数据进行分页或者排序，不能做条件判断
            JpaRepository： 继承了PagingAndSortRepository
                        开发中经常使用的接口，主要继承了PagingAndSortRepository，对返回值类型做了适配

JpaSpecificationExecutor  
提供多条件查询
```