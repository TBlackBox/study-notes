# 简介
Orika是一个快速，高效的Java Bean映射框架，主要用于在VO,PO等各种Bean之间复制属性，并且是深复制，apache commons 里面的BeanUtls使用的是反射，Orika是使用代码生成进行复制的，其性能好于BeanUtls和Dozer(使用反射，对反射信息做了缓存)。


# 文档地址
[文档地址](https://www.baeldung.com/orika-mapping)

# 引入
maven引入
```
<dependency>
    <groupId>ma.glasnost.orika</groupId>
    <artifactId>orika-core</artifactId>
    <version>1.4.6</version>
</dependency>
```

# 案列
创建两个实例，供后面的列子使用
```
public class Admin {
	private Integer id;
	private String name;
	private Integer age;
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Integer getAge() {
		return age;
	}
	public void setAge(Integer age) {
		this.age = age;
	}

	@Override
	public String toString() {
		return "Admin [id=" + id + ", name=" + name + ", age=" + age + "]";
	}
}
```
```
public class User {

	private String name;
	private Integer age;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Integer getAge() {
		return age;
	}
	public void setAge(Integer age) {
		this.age = age;
	}
	@Override
	public String toString() {
		return "User [name=" + name + ", age=" + age + "]";
	}
}

```



1. 相同实体的拷贝
```
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
MapperFacade mapperFacade = mapperFactory.getMapperFacade();

User user = new User();
user.setName("老王");
user.setAge(52);

User copyUser = mapperFacade.map(user, User.class);
System.out.println(user);
System.out.println(copyUser);
System.out.println(user.hashCode());
System.out.println(copyUser.hashCode());
```
输出：
```
User [name=老王, age=52]
User [name=老王, age=52]
555826066
174573182
```

2. 不同实例不同属性拷贝
```
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
mapperFactory.classMap(User.class, Admin.class)
.field("age", "id")
.byDefault()
.register();
MapperFacade mapperFacade = mapperFactory.getMapperFacade();

User user = new User();
user.setName("老王");
user.setAge(52);

Admin admin = mapperFacade.map(user, Admin.class);
System.out.println(user);
System.out.println(admin);
System.out.println(user.hashCode());
System.out.println(admin.hashCode());
```
输出
```
User [name=老王, age=52]
Admin [id=52, name=老王, age=null]
1558712965
2025864991
```

3. 不同实例 相同属性拷贝
```
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
		
mapperFactory.classMap(User.class, Admin.class);
MapperFacade mapperFacade = mapperFactory.getMapperFacade();

User user = new User();
user.setName("王麻子");
user.setAge(25);

Admin admin = mapperFacade.map(user, Admin.class);
```