
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