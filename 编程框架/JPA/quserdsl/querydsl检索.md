
# 基本检索
```
QUser quser = QUser.user;

//单条件查询当个字段
String userAddr = jPAQueryFactory.select(quser.addr)
    .from(quser)
    .where(quser.id.eq(1))
    .fetchOne();

//单条件查询单个对象
User user = jPAQueryFactory.select(quser)
    .from(quser)
    .where(quser.id.eq(1))
    .fetchOne();

//多条件查询
//说明：where里面的多个 参数默认是and
//下面的条件这样写也成立

//BooleanExpression ex1 = quser.id.eq(1);
//BooleanExpression ex = quser.appId.eq("2112");
//.where(ex1,ex)
User user1 = jPAQueryFactory.select(quser)
    .from(quser)
    .where(quser.id.eq(1),quser.appId.eq("2112"))
    .fetchOne();

BooleanExpression ex1 = quser.id.eq(1).or(quser.id.eq(2));
BooleanExpression ex = quser.appId.eq("2112");
User user2 = jPAQueryFactory.select(quser)
    .from(quser)
    .where(ex1,ex)
    .fetchFirst(); //返回第一个



//查询列表
//select里面的类型是什么  列表里面的类型就是什么
List<String> addrList = jPAQueryFactory.select(quser.addr)
    .from(quser)
    .fetch();
//多个字段检索在Tuple
List<Tuple> listTuple = jPAQueryFactory.select(quser.id,quser.addr)
    .from(quser)
    .fetch();
//获取值
Integer id = listTuple.get(0).get(quser.id);

//多个字段封装到实体
QBean<User> qbean = Projections.bean(User.class, quser.id,quser.addr);
List<User> userList = jPAQueryFactory.select(qbean)
    .from(quser)
    .fetch();

//多表查询
//from 里面不同的源都可以了
QBean<User> qbean1 = Projections.bean(User.class, quser.id,quser.addr);
List<Tuple> listTuple1 = jPAQueryFactory.select(QAdmin.admin,qbean1)
    .from(QAdmin.admin,quser)
    .fetch();

//一张表多次检索
//需要创建多个检索对象, 一个检索对象表示一张表
QUser user3 = new QUser("user1");
User user4 = jPAQueryFactory.select(user3)
    .from(quser)
    .innerJoin(user3).on(quser.id.eq(user3.id))
    .fetchOne();
//子查询
QAdmin qadmin = QAdmin.admin;
jPAQueryFactory.selectFrom(quser)
    .where(quser.id.in(JPAExpressions.select(qadmin.id).from(qadmin)))
    .fetch();
```

# 分页查询
```
QUser quser = QUser.user;
QueryResults<User> results = jPAQueryFactory.select(quser)
    .from(quser)
    .offset(1)
    .limit(1)
    .fetchResults();
```

结果集`QueryResults`包含下面属性
```
private final long limit, offset, total;
private final List<T> results;
```
注意：
		offset(1)	设置limit
		limit(10)	设置每页显示的数量

		* offset 不是页码, offset = (页码 - 1) * limit
		* 只要是使用了 fetchResults(), 就会进行总数量的统计查询

## 只做统计count
```
Long total = jPAQueryFactory.select(quser)
    .from(quser)
    .offset(1)
    .limit(1)
    .fetchCount();
```
## 只分页不统计
```
List<User> userList = jPAQueryFactory.select(quser)
    .from(quser)
    .offset(1)
    .limit(1)
    .fetch();
```

# 排序

## 通过属性排序
```
QUser quser = QUser.user;
jPAQueryFactory.selectFrom(QUser.user)
    .orderBy(quser.id.desc(),quser.appId.asc())
    .fetch();
		
```

## 通过构造方法排序
```
jPAQueryFactory.selectFrom(QUser.user)
        .orderBy(new OrderSpecifier(Order.ASC,quser.id,NullHandling.Default),quser.appId.asc())
.fetch();
```
`OrderSpecifier`有两个构造函数
```
public OrderSpecifier(Order order, Expression<T> target, NullHandling nullhandling) {}
public OrderSpecifier(Order order, Expression<T> target) {}
```
说明：
```
Order 枚举
	ASC
	DESC
target 排序的字段，也就是对象的属性

NullHandling 枚举
	Default 默认
	NullsFirst  null记录排前面
	NullsLast  null记录排后面
```

## 加锁
```
QUser quser = QUser.user;
jPAQueryFactory.selectFrom(QUser.user)
    .setLockMode(LockModeType.OPTIMISTIC)
    .fetch();
```
`LockModeType`枚举
```
    /**
     *  Synonymous with <code>OPTIMISTIC</code>.
     *  <code>OPTIMISTIC</code> is to be preferred for new
     *  applications.
     */
    READ,

    /**
     *  Synonymous with <code>OPTIMISTIC_FORCE_INCREMENT</code>.
     *  <code>OPTIMISTIC_FORCE_IMCREMENT</code> is to be preferred for
     *
     */
    WRITE,

    /**
     * Optimistic lock.
     */
    OPTIMISTIC,

    /**
     * Optimistic lock, with version update.
     */
    OPTIMISTIC_FORCE_INCREMENT,

    /**
     * Pessimistic read lock.
     */
    PESSIMISTIC_READ,

    /**
     * Pessimistic write lock.
     */
    PESSIMISTIC_WRITE,

    /**
     * Pessimistic write lock, with version update.
     */
    PESSIMISTIC_FORCE_INCREMENT,

    /**
     * No lock.
     */
    NONE
```

## 例子

1. 通过`BooleanBuilder`构建查询条件，BooleanBuilder 返回`Predicate`

```
public Optional<T> findOne(Predicate predicate) {
	return this.baseRepository.findOne(predicate);
}
```

```
this.findOne(new BooleanBuilder(QAdmin.admin.account.eq(account))).orElseThrow(()-> new NotFoundException("改账号不存在").setErrorData(account));
```

