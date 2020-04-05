# Spring Data JPA 简介
Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据库的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！

# Spring Data JPA作用
SpringData Jpa 极大简化了数据库访问层代码。 
如何简化的呢？ 
使用了SpringDataJpa，我们的dao层中只需要写接口，就自动具有了增删改查、分页查询等方法。


# Spring Data JPA 与 JPA和hibernate之间的关系
JPA是一套规范，内部是有接口和抽象类组成的。hibernate是一套成熟的ORM框架，而且Hibernate实现了JPA规范，所以也可以称hibernate为JPA的一种实现方式，我们使用JPA的API编程，意味着站在更高的角度上看待问题（面向接口编程）

Spring Data JPA是Spring提供的一套对JPA操作更加高级的封装，是在JPA规范下的专门用来进行数据持久化的解决方案。
可以通过下面的结构理解
```
java运用程序-->spring data jpa（对jpa的更高级封装）----->jpa(规范)--->hibernate-->数据库
```

