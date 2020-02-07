# 简介
```
多对多操作
		案例：用户和角色（多对多关系）
			用户：
			角色：
	
		分析步骤
			1.明确表关系
				多对多关系
			2.确定表关系（描述 外键|中间表）
				中间间表
			3.编写实体类，再实体类中描述表关系（包含关系）
				用户：包含角色的集合
				角色：包含用户的集合
			4.配置映射关系
			
	多表的查询
		1.对象导航查询
			查询一个对象的同时，通过此对象查询他的关联对象
			
			案例：客户和联系人
			
			从一方查询多方
				* 默认：使用延迟加载（****）
				
			从多方查询一方
				* 默认：使用立即加载
```
注意：多对多关系需要有一个中间表

# 案列
这里以用户和角色为列，一个用户有多个角色，一个角色有多个用户

## 创建符合Spring Data Jpa的spring Maven工程

## 创建实体
用户实体
```
@Entity
@Table(name = "sys_user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="user_id")
    private Long userId;
    @Column(name="user_name")
    private String userName;
    @Column(name="age")
    private Integer age;

    /**
     * 配置用户到角色的多对多关系
     *      配置多对多的映射关系
     *          1.声明表关系的配置
     *              @ManyToMany(targetEntity = Role.class)  //多对多
     *                  targetEntity：代表对方的实体类字节码
     *          2.配置中间表（包含两个外键）
     *                @JoinTable
     *                  name : 中间表的名称
     *                  joinColumns：配置当前对象在中间表的外键
     *                      @JoinColumn的数组
     *                          name：外键名
     *                          referencedColumnName：参照的主表的主键名
     *                  inverseJoinColumns：配置对方对象在中间表的外键
     */
    @ManyToMany(targetEntity = Role.class,cascade = CascadeType.ALL)
    @JoinTable(name = "sys_user_role",
            //joinColumns,当前对象在中间表中的外键
            joinColumns = {@JoinColumn(name = "sys_user_id",referencedColumnName = "user_id")},
            //inverseJoinColumns，对方对象在中间表的外键
            inverseJoinColumns = {@JoinColumn(name = "sys_role_id",referencedColumnName = "role_id")}
    )
    private Set<Role> roles = new HashSet<>();
    
    //此处省略了set get toString 方法
}

```
角色实体
```

@Entity
@Table(name = "sys_role")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "role_id")
    private Long roleId;
    @Column(name = "role_name")
    private String roleName;

    //配置多对多
    @ManyToMany(mappedBy = "roles")  //配置多表关系
    private Set<User> users = new HashSet<>();

    //此处省略了set get toString 方法
}

```

## 编写到接口
用户dao
```
public interface UserDao extends JpaRepository<User,Long> ,JpaSpecificationExecutor<User> {
}

```
角色dao
```
public interface RoleDao extends JpaRepository<Role,Long> ,JpaSpecificationExecutor<Role> {
}

```

## 测试案列
```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class ManyToManyTest {

    @Autowired
    private UserDao userDao;
    @Autowired
    private RoleDao roleDao;

    /**
     * 保存一个用户，保存一个角色
     *
     *  多对多放弃维护权：被动的一方放弃
     */
    @Test
    @Transactional
    @Rollback(false)
    public void  testAdd() {
        User user = new User();
        user.setUserName("小李");

        Role role = new Role();
        role.setRoleName("java程序员");

        //配置用户到角色关系，可以对中间表中的数据进行维护     1-1
        user.getRoles().add(role);

        //配置角色到用户的关系，可以对中间表的数据进行维护     1-1
        role.getUsers().add(user);

        userDao.save(user);
        roleDao.save(role);
    }


    //测试级联添加（保存一个用户的同时保存用户的关联角色）
    @Test
    @Transactional
    @Rollback(false)
    public void  testCasCadeAdd() {
        User user = new User();
        user.setUserName("小李");

        Role role = new Role();
        role.setRoleName("java程序员");

        //配置用户到角色关系，可以对中间表中的数据进行维护     1-1
        user.getRoles().add(role);

        //配置角色到用户的关系，可以对中间表的数据进行维护     1-1
        role.getUsers().add(user);

        userDao.save(user);
    }

    /**
     * 案例：删除id为1的用户，同时删除他的关联对象
     */
    @Test
    @Transactional
    @Rollback(false)
    public void  testCasCadeRemove() {
        //查询1号用户
        User user = userDao.findOne(1l);
        //删除1号用户
        userDao.delete(user);

    }
}

```
## 注解说明
@ManyToMany
	作用：用于映射多对多关系
	属性：
		cascade：配置级联操作。
		fetch：配置是否采用延迟加载。
    	targetEntity：配置目标的实体类。映射多对多的时候不用写。

@JoinTable
    作用：针对中间表的配置
    属性：
    	nam：配置中间表的名称
    	joinColumns：中间表的外键字段关联当前实体类所对应表的主键字段			  			
    	inverseJoinColumn：中间表的外键字段关联对方表的主键字段
    	
@JoinColumn
    作用：用于定义主键字段和外键字段的对应关系。
    属性：
    	name：指定外键字段的名称
    	referencedColumnName：指定引用主表的主键字段名称
    	unique：是否唯一。默认值不唯一
    	nullable：是否允许为空。默认值允许。
    	insertable：是否允许插入。默认值允许。
    	updatable：是否允许更新。默认值允许。
    	columnDefinition：列的定义信息。
    	
## 注意的地方
在多对多（保存）中，如果双向都设置关系，意味着双方都维护中间表，都会往中间表插入数据，中间表的2个字段又作为联合主键，所以报错，主键重复，解决保存失败的问题：只需要在任意一方放弃对中间表的维护权即可，推荐在被动的一方放弃，配置如下：
```
	//放弃对中间表的维护权，解决保存中主键冲突的问题
	@ManyToMany(mappedBy="roles")
	private Set<SysUser> users = new HashSet<SysUser>(0);
```
# 总结
这里只是一个 简单的列子，方便自己忘记的时候能够快速的回忆起。对象导航查询在一对多的案列中。

