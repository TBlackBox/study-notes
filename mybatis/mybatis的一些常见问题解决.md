
# mybatis   获取自增id

1. 第一种方式
在配置文件里面加 `useGeneratedKeys="true"` `keyProperty="id"`
```
<insert id="insertMessage" useGeneratedKeys="true" keyProperty="id"  parameterType="com.customer.system.customer.entity.MessageEntity">
    INSERT INTO `message`(`type`,`message_type`,`state`,`from_id`,`to_id`,`content`,`create_date`)
    VALUES(#{type},#{messageType},#{state},#{fromId},#{toId},#{content},#{createDate})
</insert>
```

2. 使用SelectKey返回主键的值

```
<insert id="insertSysUser" parameterType="SysUser"  >
    insert into sys_user(user_name,password,user_info,head_img,create_time) values(
    #{userName},#{password},#{userInfo},#{headImg},#{createTime});
    <selectKey keyColumn="id" keyProperty="id" resultType="long" order="AFTER">
        SELECT LAST_INSERT_ID ()
    </selectKey>
</insert>

```


3. 通过 注解
在insert  方法上添加注解
```
@Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id")
```


# mybatis复杂数据查询

## 多对多关系
描述：
查询多条数据，每条数据的某个字段是又是列表(集合),可以使用mybatis的集合查询。
```
<resultMap id="result_01" type="com.video.common.domain.model.AdminModel">
		<id column="id" property="id"/>
		<id column="name" property="name"/>
		<id column="account" property="account"/>
        <!--property 实体类对应的属性名称  ofType 集合里面对应的类型  columnPrefix column 的前缀，可以不写，就需要写在column上面，方便区分-->
		<collection property="roles" ofType="com.video.common.domain.entity.RoleEntity" columnPrefix="role_">
			<id column="id" property="id"/>
			<id column="name" property="name"/>
		</collection>
    </resultMap>
	
	<select id="selectList" resultMap="result_01" >
		SELECT 
			`a`.`id`,
			`a`.`name`,
			`a`.`account`,
			
			`ar`.`id` AS `role_id`,
			`ar`.`name` AS `role_name`
		FROM 
			`admin` AS `a`
			INNER JOIN `auth_admin_role` AS `aar` ON `a`.`id` = `aar`.`admin_id`
			INNER JOIN `auth_role` AS `ar` ON `aar`.`role_id` = `ar`.`id`
	</select>
``` 
说明：`AdminModel`里面有`id`,`name`,`account`属性，还包含一个`List<RoleEntity> roles`的属性, `RoleEntity`里面包含`roleId`,`roleName`等属性，即一个用户对应多个角色的查询，又会查询出多个用户。mybatis会让id作为一个建，一样的id的话，就会把相应的角色信息放到集合里面。