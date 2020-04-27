# JPA和Hibernate的关系
JPA（Java Persistence API），是Java EE 5的标准ORM接口，也是ejb3规范的一部分。

Hibernate是之前很流行的ORM框架，也是JPA的一个实现，其它还有Toplink之类的ROM框架。

```
JPA和Hibernate之间的关系，可以简单的理解为JPA是标准接口，Hibernate是实现。

```

Hibernate主要是通过三个组件来实现的：

- hibernate-core：Hibernate的核心实现，提供了Hibernate所有的核心功能。
- hibernate-entitymanager：Hibernate实现了标准的JPA，可以把它看成hibernate-core和JPA之间的适配器，它并不直接提供ORM的功能，而是对hibernate-core进行封装，使得Hibernate符合JPA的规范。
- hibernate-annotation：Hibernate支持annotation方式配置的基础，它包括了标准的JPA annotation以及Hibernate自身特殊功能的annotation。


# 注解说明
Hibernate的注解在`org.hibernate.annotations`包下，JPA的注释在`javax.persistence`包下。

# Entity
```
@Entity说明这个class是实体类，并且使用默认的orm规则，即class名对应数据库表中表名，class字段名即表中的字段名。
（如果想改变这种默认的orm规则，就要使用@Table来改变class名与数据库中表名的映射规则，@Column来改变class中字段名与db中表的字段名的映射规则）
```
元数据属性说明：
• name: 表名

下面的代码说明,Customer类对应数据库中的Customer表，其中name为可选，缺省类名即表名！
```
    @Entity(name=”Customer”)
    public class Customer { ... }
```

# Table
```
Table用来定义entity主表的name，catalog，schema等属性。
```
元数据属性说明：
-  name: 表名
- catalog: 对应关系数据库中的catalog
- schema：对应关系数据库中的schema
- UniqueConstraints:定义一个UniqueConstraint数组，指定需要建唯一约束的列
- indexes：定义一个索引数组，指定索引的列
```
@Entity
@Table(name="user",indexes = {@Index(columnList = "rid")},uniqueConstraints = {@UniqueConstraint(columnNames={"app_id", "role"})})
public class User { ... }
```

*** 注意：***
Heibernate的注解`comment`可以给表添加注释
```
@Entity
@Table(name = "user")
@org.hibernate.annotations.Table(appliesTo = "user",comment = "用户表")
public class User {}
```

# SecondaryTable
```
一个entity class可以映射到多表，SecondaryTable用来定义单个从表的名字，主键名字等属性。
```
元数据属性说明：
• name: 表名
• catalog: 对应关系数据库中的catalog
• schema：对应关系数据库中的schema
• pkJoin: 定义一个PrimaryKeyJoinColumn数组，指定从表的主键列
• UniqueConstraints: 定义一个UniqueConstraint数组，指定需要建唯一约束的列

下面的代码说明Customer类映射到两个表，主表名是CUSTOMER，从表名是CUST_DETAIL，从表的主键列和主表的主键列类型相同，列名为CUST_ID。

	@Entity
	@Table(name="CUSTOMER")
	@SecondaryTable(name="CUST_DETAIL",pkJoin=@PrimaryKeyJoinColumn(name="CUST_ID"))
	public class Customer { ... }

# SecondaryTables
当一个entity class映射到一个主表和多个从表时，用SecondaryTables来定义各个从表的属性。

元数据属性说明：
• value： 定义一个SecondaryTable数组，指定每个从表的属性。

	@Table(name = "CUSTOMER")
	@SecondaryTables( value = {
	@SecondaryTable(name = "CUST_NAME", pkJoin = { @PrimaryKeyJoinColumn(name = "STMO_ID", referencedColumnName = "id") }),
	@SecondaryTable(name = "CUST_ADDRESS", pkJoin = { @PrimaryKeyJoinColumn(name = "STMO_ID", referencedColumnName = "id") }) })
	public class Customer {}

# UniqueConstraint
```
UniqueConstraint定义在Table或SecondaryTable元数据里，用来指定建表时需要建唯一约束的列。
```
元数据属性说明：
• columnNames:定义一个字符串数组，指定要建唯一约束的列名。

		@Entity
		@Table(name="EMPLOYEE",
			uniqueConstraints={@UniqueConstraint(columnNames={"EMP_ID", "EMP_NAME"})}
		)
		public class Employee { ... }

# Column
```
Column元数据定义了映射到数据库的列的所有属性：列名，是否唯一，是否允许为空，是否允许更新等。
```
元数据属性说明：
• name:列名。
• unique: 是否唯一
• nullable: 是否允许为空
• insertable: 是否允许插入
• updatable: 是否允许更新
• columnDefinition: 定义建表时创建此列的DDL
• secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字。
```
    public class Person {
    @Column(name = "PERSONNAME", unique = true, nullable = false, updatable = true)
    private String name;
    
    @Column(name = "PHOTO", columnDefinition = "BLOB NOT NULL", secondaryTable="PER_PHOTO")
    private byte[] picture;
```

# OneToOne
```
@OneToOne描述一个 一对一的关联
```
元数据属性说明：
• fetch：表示抓取策略，默认为FetchType.LAZY
• cascade：表示级联操作策略
```
@OneToOne(fetch=FetchType,cascade=CascadeType)
```
# ManyToOne
```
@ManyToOne表示一个多对一的映射,该注解标注的属性通常是数据库表的外键.
```
元数据属性说明：
• optional：是否允许该字段为null，该属性应该根据数据库表的外键约束来确定，默认为true
• fetch：表示抓取策略，默认为FetchType.EAGER
• cascade：表示默认的级联操作策略，可以指定为ALL，PERSIST，MERGE，REFRESH和REMOVE中的若干组合，默认为无级联操作
• targetEntity：表示该属性关联的实体类型。该属性通常不必指定，ORM框架根据属性类型自动判断targetEntity。
```
@ManyToOne(fetch=FetchType,cascade=CascadeType)
```
# OneToMany
```
@OneToMany描述一个 一对多的关联,该属性应该为集体类型,在数据库中并没有实际字段。
```
元数据属性说明：
• fetch：表示抓取策略,默认为FetchType.LAZY,因为关联的多个对象通常不必从数据库预先读取到内存
• cascade：表示级联操作策略,对于OneToMany类型的关联非常重要,通常该实体更新或删除时,其关联的实体也应当被更新或删除

例如：
实体User和Order是OneToMany的关系，则实体User被删除时，其关联的实体Order也应该被全部删除
```
@OneToMany(fetch=FetchType,cascade=CascadeType)
```

# ManyToMany
@ManyToMany 描述一个多对多的关联.多对多关联上是两个一对多关联,但是在ManyToMany描述中,中间表是由ORM框架自动处理

元数据属性说明：
• targetEntity:表示多对多关联的另一个实体类的全名,例如:package.Book.class
• mappedBy:表示多对多关联的另一个实体类的对应集合属性名称

两个实体间相互关联的属性必须标记为@ManyToMany,并相互指定targetEntity属性,
需要注意的是,有且只有一个实体的@ManyToMany注解需要指定mappedBy属性,指向targetEntity的集合属性名称
利用ORM工具自动生成的表除了User和Book表外,还自动生成了一个User_Book表,用于实现多对多关联

# JoinColumn
```
如果在entity class的field上定义了关系（one2one或one2many等），我们通过JoinColumn来定义关系的属性。JoinColumn的大部分属性和Column类似。
```
元数据属性说明：
• name:列名。
• referencedColumnName:该列指向列的列名（建表时该列作为外键列指向关系另一端的指定列）
• unique: 是否唯一
• nullable: 是否允许为空
• insertable: 是否允许插入
• updatable: 是否允许更新
• columnDefinition: 定义建表时创建此列的DDL
• secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字。

下面的代码说明Custom和Order是一对一关系。在Order对应的映射表建一个名为CUST_ID的列，该列作为外键指向Custom对应表中名为ID的列。
```
	public class Custom {
	
	@OneToOne
	@JoinColumn(
	name="CUST_ID", referencedColumnName="ID", unique=true, nullable=true, updatable=true)
	public order getOrder() {
		return order;
	}
```
# JoinColumns
```
如果在entity class的field上定义了关系（one2one或one2many等），并且关系存在多个JoinColumn，用JoinColumns定义多个JoinColumn的属性。
```
元数据属性说明：
• value: 定义JoinColumn数组，指定每个JoinColumn的属性。

下面的代码说明Custom和Order是一对一关系。在Order对应的映射表建两列，一列名为CUST_ID，该列作为外键指向Custom对应表中名为ID的列,另一列名为CUST_NAME，该列作为外键指向Custom对应表中名为NAME的列。
```
	public class Custom {
	@OneToOne
	@JoinColumns({
	@JoinColumn(name="CUST_ID", referencedColumnName="ID"),
	@JoinColumn(name="CUST_NAME", referencedColumnName="NAME")
	})
	public order getOrder() {
		return order;
	}
```
# Id
```
声明当前field为映射表中的主键列。

```
id值的获取方式有五种：`TABLE`, `SEQUENCE`, `IDENTITY`, `AUTO`, `NONE`。
1. Oracle和DB2支持`SEQUENCE`
2. SQL Server和Sybase支持`IDENTITY`
3. MySQL支持`AUTO`。
所有的数据库都可以指定为AUTO，我们会根据不同数据库做转换。
`NONE` (默认)需要用户自己指定Id的值。

元数据属性说明：
• generate():主键值的获取类型
• generator():TableGenerator的名字（当generate=GeneratorType.TABLE才需要指定该属性）

下面的代码声明Task的主键列id是自动增长的。(Oracle和DB2从默认的SEQUENCE取值，SQL Server和Sybase该列建成IDENTITY，mysql该列建成auto increment。)
```
    @Entity
    @Table(name = "OTASK")
    public class Task {
      @Id(generate = GeneratorType.AUTO)
      public Integer getId() {
    	  return id;
      }
    }
```

# IdClass
```
当entity class使用复合主键时，需要定义一个类作为id class。
```
## IdClass必须符合下面3个条件
1. 类必须声明为public，并提供一个声明为public的空构造函数。
2. 必须实现Serializable接口，覆写 equals() 和 hashCode()方法。
3. entity class的所有id field在id class都要定义，且类型一样。

元数据属性说明：
• value: id class的类名
```
    public class EmployeePK implements java.io.Serializable{
       String empName;
       Integer empAge;
    
       public EmployeePK(){}
    
       public boolean equals(Object obj){ ......}
       public int hashCode(){......}
    }


    @IdClass(value=com.acme.EmployeePK.class)
    @Entity(access=FIELD)
    public class Employee {
        @Id String empName;
        @Id Integer empAge;
    }
```

# MapKey
```
在一对多，多对多关系中，我们可以用Map来保存集合对象。默认用主键值做key，如果使用复合主键，则用id class的实例做key，如果指定了name属性，就用指定的field的值做key。
```
元数据属性说明：
• name: 用来做key的field名字

下面的代码说明Person和Book之间是一对多关系。Person的books字段是Map类型，用Book的isbn字段的值作为Map的key。
```
    @Table(name = "PERSON")
    public class Person {
    
    @OneToMany(targetEntity = Book.class, cascade = CascadeType.ALL, mappedBy = "person")
    @MapKey(name = "isbn")
    private Map books = new HashMap();
    }
```

# OrderBy
```
在一对多，多对多关系中，有时我们希望从数据库加载出来的集合对象是按一定方式排序的，这可以通过OrderBy来实现，默认是按对象的主键升序排列。
```
元数据属性说明：
• value: 字符串类型，指定排序方式。
格式为"fieldName1 [ASC|DESC],fieldName2 [ASC|DESC],…",排序类型可以不指定，默认是ASC。

下面的代码说明Person和Book之间是一对多关系。集合books按照Book的isbn升序，name降序排列。
```
    @Table(name = "MAPKEY_PERSON")
    public class Person {
    
    @OneToMany(targetEntity = Book.class, cascade = CascadeType.ALL, mappedBy = "person")
    @OrderBy(name = "isbn ASC, name DESC")
    private List books = new ArrayList();
    }
```
# PrimaryKeyJoinColumn
在三种情况下会用到PrimaryKeyJoinColumn。
```
• 继承。
• entity class映射到一个或多个从表。从表根据主表的主键列（列名为referencedColumnName值的列），建立一个类型一样的主键列，列名由name属性定义。
• one2one关系，关系维护端的主键作为外键指向关系被维护端的主键，不再新建一个外键列。
```
元数据属性说明：
• name:列名。
• referencedColumnName:该列引用列的列名
• columnDefinition: 定义建表时创建此列的DDL

下面的代码说明Customer映射到两个表，主表CUSTOMER,从表CUST_DETAIL，从表需要建立主键列CUST_ID，该列和主表的主键列id除了列名不同，其他定义一样。
````
    @Entity
    @Table(name="CUSTOMER")
    @SecondaryTable(name="CUST_DETAIL",pkJoin=@PrimaryKeyJoinColumn(name="CUST_ID"，referencedColumnName="id"))
    public class Customer { 
     @Id(generate = GeneratorType.AUTO)
      public Integer getId() {
    	  return id;
      }
    }
````
下面的代码说明Employee和EmployeeInfo是一对一关系，Employee的主键列id作为外键指向EmployeeInfo的主键列INFO_ID。
```
    @Table(name = "Employee")
    public class Employee {
    @OneToOne
    @PrimaryKeyJoinColumn(name = "id", referencedColumnName="INFO_ID")
    EmployeeInfo info;
    }
```
# PrimaryKeyJoinColumns
```
如果entity class使用了复合主键，指定单个PrimaryKeyJoinColumn不能满足要求时，可以用PrimaryKeyJoinColumns来定义多个PrimaryKeyJoinColumn。
```
元数据属性说明：
• value: 一个PrimaryKeyJoinColumn数组，包含所有PrimaryKeyJoinColumn。

下面的代码说明了Employee和EmployeeInfo是一对一关系。他们都使用复合主键，建表时需要在Employee表建立一个外键，从Employee的主键列id,name指向EmployeeInfo的主键列INFO_ID和INFO_NAME.

```
   @Entity
   @IdClass(EmpPK.class)
   @Table(name = "EMPLOYEE")
   public class Employee {

	private int id;
	
	private String name;
	
	private String address;
	
	@OneToOne(cascade = CascadeType.ALL)
	@PrimaryKeyJoinColumns({
	@PrimaryKeyJoinColumn(name="id", referencedColumnName="INFO_ID"),
	@PrimaryKeyJoinColumn(name="name" , referencedColumnName="INFO_NAME")})
	EmployeeInfo info;
	}
	
	@Entity
	@IdClass(EmpPK.class)
	@Table(name = "EMPLOYEE_INFO")
	public class EmployeeInfo {
	
	@Id
	@Column(name = "INFO_ID")
	private int id;
	
	@Id
	@Column(name = "INFO_NAME")
	private String name;
	}
```
# Transient
```
@Transient表示该属性并不是一个到数据库表的字段的映射，指定的这些属性不会被持久化，ORM框架将忽略该属性。
如果一个属性并非数据库表的字段映射。就务必将其标示为@Transient。否则，ORM框架默认其注解为@Basic
```
```
    @Transient
    private String name;
```
# Version
```
Version指定实体类在乐观事务中的version属性。在实体类重新由EntityManager管理并且加入到乐观事务中时，保证完整性。每一个类只能有一个属性被指定为version，version属性应该映射到实体类的主表上。
```
下面的代码说明versionNum属性作为这个类的version，映射到数据库中主表的列名是OPTLOCK。
```
    @Version
    @Column("OPTLOCK")
    protected int getVersionNum() { return versionNum; }
```
# Lob
```
Lob指定一个属性作为数据库支持的大对象类型在数据库中存储。使用LobType这个枚举来定义Lob是二进制类型还是字符类型。
```
LobType枚举类型说明：
• BLOB 二进制大对象，Byte[]或者Serializable的类型可以指定为BLOB。
• CLOB 字符型大对象，char[]、Character[]或String类型可以指定为CLOB。

元数据属性说明：
• fetch： 定义这个字段是lazy loaded还是eagerly fetched。数据类型是FetchType枚举，默认为LAZY,即lazy loaded.
• type： 定义这个字段在数据库中的JDBC数据类型。数据类型是LobType枚举，默认为BLOB。

下面的代码定义了一个BLOB类型的属性和一个CLOB类型的属性。
```
    @Lob
    @Column(name="PHOTO" columnDefinition="BLOB NOT NULL")
    protected JPEGImage picture;
    
    @Lob(fetch=EAGER, type=CLOB)
    @Column(name="REPORT")
    protected String report;
```

# JoinTable
```
JoinTable在many-to-many关系的所有者一边定义。如果没有定义JoinTable，使用JoinTable的默认值。
```
元数据属性说明：
• table:这个join table的Table定义。
• joinColumns:定义指向所有者主表的外键列，数据类型是JoinColumn数组。
• inverseJoinColumns:定义指向非所有者主表的外键列，数据类型是JoinColumn数组。

下面的代码定义了一个连接表CUST和PHONE的join table。join table的表名是CUST_PHONE，包含两个外键，一个外键是CUST_ID，指向表CUST的主键ID，另一个外键是PHONE_ID，指向表PHONE的主键ID。
```
    @JoinTable(
    table=@Table(name=CUST_PHONE),
    joinColumns=@JoinColumn(name="CUST_ID", referencedColumnName="ID"),
    inverseJoinColumns=@JoinColumn(name="PHONE_ID", referencedColumnName="ID")
    )
```
# TableGenerator
```
TableGenerator定义一个主键值生成器，在Id这个元数据的generate＝TABLE时，generator属性中可以使用生成器的名字。生成器可以在类、方法或者属性上定义。
生成器是为多个实体类提供连续的ID值的表，每一行为一个类提供ID值，ID值通常是整数。
```
元数据属性说明：
• name:生成器的唯一名字，可以被Id元数据使用。
• table:生成器用来存储id值的Table定义。
• pkColumnName:生成器表的主键名称。
• valueColumnName:生成器表的ID值的列名称。
• pkColumnValue:生成器表中的一行数据的主键值。
• initialValue:id值的初始值。
• allocationSize:id值的增量。

下面的代码定义了两个生成器empGen和addressGen，生成器的表是ID_GEN。
```
    @Entity
    public class Employee {
    ...
    @TableGenerator(name="empGen",
    table=@Table(name="ID_GEN"),
    pkColumnName="GEN_KEY",
    valueColumnName="GEN_VALUE",
    pkColumnValue="EMP_ID",
    allocationSize=1)
    @Id(generate=TABLE, generator="empGen")
    public int id;
    ...
    }
    
    @Entity 
    public class Address {
    ...
    @TableGenerator(name="addressGen",
    table=@Table(name="ID_GEN"),
    pkColumnValue="ADDR_ID")
    @Id(generate=TABLE, generator="addressGen")
    public int id;
    ...
    }
```
# SequenceGenerator
```
SequenceGenerator定义一个主键值生成器，在Id这个元数据的generator属性中可以使用生成器的名字。生成器可以在类、方法或者属性上定义。生成器是数据库支持的sequence对象。
```
元数据属性说明：
• name:生成器的唯一名字，可以被Id元数据使用。
• sequenceName:数据库中，sequence对象的名称。如果不指定，会使用提供商指定的默认名称。
• initialValue:id值的初始值。
• allocationSize:id值的增量。

下面的代码定义了一个使用提供商默认名称的sequence生成器。
```
    @SequenceGenerator(name="EMP_SEQ", allocationSize=25)	
```
# DiscriminatorColumn
```
DiscriminatorColumn定义在使用SINGLE_TABLE或JOINED继承策略的表中区别不继承层次的列。
```
元数据属性说明：
• name:column的名字。默认值为TYPE。
• columnDefinition:生成DDL的sql片断。
• length:String类型的column的长度，其他类型使用默认值10。

下面的代码定义了一个列名为DISC，长度为20的String类型的区别列。
```
    @Entity
    @Table(name="CUST")
    @Inheritance(strategy=SINGLE_TABLE,
        discriminatorType=STRING,
       discriminatorValue="CUSTOMER")
    @DiscriminatorColumn(name="DISC", length=20)
    public class Customer { ... }
```
# Hibernate验证注解
|注解|	适用类型|	说明|	示例 |
|------|-------|------|------|
|@Pattern	|String	|通过正则表达式来验证字符串|	@attern(regex=”[a-z]{6}”)|
|@Length	|String	|验证字符串的长度|	@length(min=3,max=20)|
|@Email	|String	|验证一个Email地址是否有效	|@email|
|@Range	|Long	|验证一个整型是否在有效的范围内|	@Range(min=0,max=100)|
|@Min	|Long	|验证一个整型必须不小于指定值|	@Min(value=10)|
|@Max	|Long	|验证一个整型必须不大于指定值|	@Max(value=20)|
|@Size|	集合或数组|	集合或数组的大小是否在指定范围内|	@Size(min=1,max=255)|
以上每个注解都可能性有一个message属性，用于在验证失败后向用户返回的消息|

# 总结
## @JoinColumn
@OneToOne注释只能确定实体与实体的关系是一对一的关系，不能指定数据库表中的保存的关联字段。所以此时要结合@JoinColumn标记来指定保存实体关系的配置。

@JoinColumn与上面讲述的@Column注释类似，它的定义如下代码所示。
```
@Target({METHOD, FIELD})
 @Retention(RUNTIME) 
String name() default ""; 
String referencedColumnName() default ""; 
boolean unique() default false; 
boolean nullable() default true; 
boolean insertable() default true; 
boolean updatable() default true; 
String columnDefinition() default ""; 
String table() default ""; 
} 
```
在使用@JoinColumn注释时，应注意以下几个问题。
1、 @JoinColumn与@Column标记一样，是用于注释表中的字段的。它的属性与@Column属性有很多相同之处，这里就不详细讲述。
2、@JoinColumn与@Column相区别的是：@JoinColumn注释的是保存表与表之间关系的字段，它要标注在实体属性上。而@Column标注的是表中不包含表关系的字段。
3、与@Column标记一样，name属性是用来标识表中所对应的字段的名称。

例如customer表中存在字段addr_id，标识的代码如下所示。
```
@OneToOne 
@JoinColumn(name = "addr_id") 
public AddressEO getAddress() {
      return address; 
         }
```
若此时，不设置name的值，则在默认情况下，name的取值遵循以下规则：
name=关联表的名称+“_”+ 关联表主键的字段名

例如，CustomerEO实体中，如果不指定name的值，默认将对应name=address_id；

因为@JoinColumn注释在实体 AddressEO属性上，实体AddressEO对应的表名为“address”；表address的主键是“id”，所以此时对应的默认的字段名称为 “address_id”。

## 提示：
此规则只适用于与@OneToOne标记同时使用时。若与@ManyToOne或@ManyToMany标记同时使用时，将遵循其他的规则。

1、默认情况下，关联的实体的主键一般是用来做外键的。但如果此时不想主键作为外键，则需要设置referencedColumnName属性。

例如，将address表中增加一个字段“ref_id”，address表的建表SQL变为以下所示。
```
CREATE TABLE address (
    id int(20) NOT NULL auto_increment, 
ref_id int int(20) NOT NULL, 
province varchar(50) ,   
city varchar(50) , 
postcode varchar(50) , 
detail varchar(50) , 
PRIMARY KEY (id) 
) 
```
此时，通过customer表中的“address_id”字段关联的是address表中的“ref_id”，而“ref_id”并不是address表中的主键，则实体中标注如代码下所示。
```
@OneToOne 
@JoinColumn(name = "address_id",referencedColumnName="ref_id") 
public AddressEO getAddress() {
         return address;     
} 
```
属性referencedColumnName标注的是所关联表中的字段名，若不指定则使用的所关联表的主键字段名作为外键。

JoinColumn标记不仅能够与@OneToOne使用，也可以@ManyToOne或@ManyToMany标记同时使用，它们所表示的含义不同。
