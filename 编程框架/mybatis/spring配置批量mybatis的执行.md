
# 简介
mybatis 可以通过<foreach>标签的形式批量执行插入操作，但这样做有个弊端，sql语句不能太长。太长会导致报错。mybatis有方法解决这个问题，mybatis默认的执行器是简单的，我们可以通过type设置复杂的 批量处理类型。下面是spring的配置

1. 在spring的配置文件中加入如下配置
```
	<!--配置一个可以进行批量执行的sqlSession  -->
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg name="sqlSessionFactory" ref="sqlSessionFactoryBean"></constructor-arg>
		<constructor-arg name="executorType" value="BATCH"></constructor-arg>
	</bean>
```

2. 在需要的 地方注入`sqlSession`就可以了