# 配置步骤
1. 引入依赖
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

*** 注意：最新依赖可以看这里[最新依赖](http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)***


2. 在pom.xml里添加数据源配置
```yml
spring:
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://152.32.192.200:3306/blogdb?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2b8&allowMultiQueries=true
    username: blog
    password: privateblog
```

3. pom.xml里面添加配置文件路径和映射文件路径
```yaml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations:
    - classpath*:mapper/**/*-mapper.xml
    - classpath*:mapper/**/*-mapper-ext.xml
```
并在resources下创建mybatis和mapper文件夹

4. 在mybatis文件夹下加入配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <settings>
        <!-- 自动转换实体的属性和列名的规则（驼峰-下划线） -->
        <setting name="mapUnderscoreToCamelCase" value="true" />
        <!-- 默认枚举处理 -->
        <setting name="defaultEnumTypeHandler" value="org.apache.ibatis.type.EnumOrdinalTypeHandler"/>
        <!-- 自动映射，发现未知列（或者未知属性类型）抛出异常
        <setting name="autoMappingUnknownColumnBehavior" value="FAILING"/>
        -->
        <!-- 自动映射，任何复杂的结果集 -->
        <!-- <setting name="autoMappingBehavior" value="FULL"/>  -->
    </settings>
</configuration>
```

5. 创建mapper接口
```java
package cn.javafroum.mybatis.springboot2.mapper;

import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Map;

@Repository
public interface LogMapper {

    @Select("select * from logs")
    List<Map<String,Object>> selectLogList();

    List<Map<String,Object>> selectLogList1();
}

```

6. 在mapper文件夹下创建配置文件,并按照配置的方式命名

   **特别注意：.xml文件最好放到sources目录下，如果放在src/main/java代码里面，java打包将不会把xml文件打入会报下面的错误**

   ```
   org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): 
   ```

   如果要放在src/main/java代码里面，需要在maven进行配置

   ```xml
   <resources>
       <resource>
           <directory>src/main/java</directory>
           <includes>
               <include>**/*.xml</include>
           </includes>
       </resource>
       <resource>
           <directory>src/main/resources</directory>
       </resource>
   </resources>
   ```

   
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.javafroum.mybatis.springboot2.mapper.LogMapper">
    <select id="selectLogList1" resultType="java.util.Map">
        select * from logs
    </select>
</mapper>
```
例如：log-mapper.xml

7. 在springboot入口添加包扫描
```java
@MapperScan(basePackages = {"cn.javafroum.mybatis.springboot2.mapper"},annotationClass = Repository.class)
@SpringBootApplication
public class Springboot2Application {

    public static void main(String[] args) {
        SpringApplication.run(Springboot2Application.class, args);
    }
}
```

8. 测试
```java
@Controller
public class MybatisTest {
    
    @Autowired
    LogMapper logMapper;

    @ResponseBody
    @RequestMapping("/hello")
    public Object hello(){

        return logMapper.selectLogList1();
    }
}
```