# 简单查询
## 返回单一数据
```
@Test
public void testSelectOne() {
    QUser quser = QUser.user;
    User retUser = jPAQueryFactory.selectFrom(quser)
    .where(quser.id.eq(1))
    .fetchOne();
    System.out.println(retUser.toString());
}
```

## 返回多条数据
```
@Test
public void testSelectList() {
    QUser quser = QUser.user;
    List<User> retUserList = jPAQueryFactory.selectFrom(quser)
    .where(quser.id.eq(1))
    .fetch();
    System.out.println(retUserList.toString());
}
```

如果是多条件可以这样写：
```
//方式1
.where(quser.id.eq(1),quser.appId.eq("aa"))
//方式2
.where(quser.id.eq(1).and(quser.appId.eq("aa")).or(quser.createDate.between(LocalDateTime.now(), LocalDateTime.now().plusDays(1))))
```

多个表
```
@Test
public void testMulSources() {
    QUser quser = QUser.user;
    QAdmin qdmin = QAdmin.admin;
    jPAQueryFactory.from(quser,qdmin).fetch();
}
```

置顶字段查询
```
 List<UserAdminDao> idList = jPAQueryFactory.select(quser.id,qdmin.id).from(quser).fetch();
```

# 链接查询
内连接查询
```
@Test
public void testMulSources() {
    QUser quser = QUser.user;
    QAdmin qadmin = QAdmin.admin;
    jPAQueryFactory.selectFrom(quser)
    .innerJoin(qadmin).on(quser.id.eq(qadmin.id))
    .fetch();
}
```
同理，其他也是一样的：
左连接：leftJoin
右链接：rightJoin

# 常用语法
- select: Set the projection of the query. (Not necessary if created via query factory)

- from: Add the query sources here.

- innerJoin, join, leftJoin, rightJoin, on: Add join elements using these constructs. For the join methods the first argument is the join source and the second the target (alias).

- where: Add query filters, either in varargs form separated via commas or cascaded via the and-operator.

- groupBy: Add group by arguments in varargs form.

- having: Add having filters of the "group by" grouping as an varags array of Predicate expressions.

- orderBy: Add ordering of the result as an varargs array of order expressions. Use asc() and desc() on numeric, string and other comparable expression to access the OrderSpecifier instances.

- limit, offset, restrict: Set the paging of the result. Limit for max results, offset for skipping rows and restrict for defining both in one call.


## 排序
```
.orderBy(quser.id.asc(),quser.name.desc())
```

## 分组
```
.groupBy(quser.appId)
```

# 删除
```
jPAQueryFactory.delete(quser).where(quser.id.eq(1)).execute();
```

# 更新
```
jPAQueryFactory.update(quser).where(quser.id.eq(1)).set(quser.name, "菜鸟").execute();
```

# 子查询
```
@Test
public void testSub() {
    QUser quser = QUser.user;
    QAdmin qadmin = QAdmin.admin;
    jPAQueryFactory.selectFrom(quser)
    .where(quser.id.in(JPAExpressions.select(qadmin.id).from(qadmin)))
    .fetch();
}
```
# 调整语句
如果查询之前需要调整语句可以这样
```
Query jpaQuery = queryFactory.selectFrom(quser).createQuery();
// ...
List results = jpaQuery.getResultList();
```