# 引入maven依赖
```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```


# 配置配置文件
```yaml
spring:
   datasource:
       url: jdbc:h2:mem:h2test;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
       platform: h2
       username: root
       password: 123456
       driverClassName: org.h2.Driver
       
  h2:
       console:
         enabled: true
         path: /console
         settings:
           trace: false
           web-allow-others: false
```

# 测试

今日控制台
```
http://127.0.0.1/console
```

# 一些配置的说明
```properties
# h2配置
spring.jpa.show-sql = true #启用SQL语句的日志记录
spring.jpa.hibernate.ddl-auto = update  #设置ddl模式
##数据库连接设置
spring.datasource.url = jdbc:h2:mem:dbtest  #配置h2数据库的连接地址
spring.datasource.username = sa  #配置数据库用户名
spring.datasource.password = sa  #配置数据库密码
spring.datasource.driverClassName =org.h2.Driver  #配置JDBC Driver

几种链接方式
1. 内存模式
链接字符串 jdbc:h2:mem 或 jdbc:h2:mem:<databaseName>，如果 <databaseName> 不写，则每次创建链接都是新的数据库。
2.  文件模式
链接字符串：jdbc:h2:[file:][<path>]<databaseName>
3. 服务器模式
4. 混合模式

## 数据初始化设置
spring.datasource.schema=classpath:db/schema.sql  #进行该配置后，每次启动程序，程序都会运行resources/db/schema.sql文件，对数据库的结构进行操作。
spring.datasource.data=classpath:db/data.sql  #进行该配置后，每次启动程序，程序都会运行resources/db/data.sql文件，对数据库的数据操作。

##h2 web console设置
spring.datasource.platform=h2  #表明使用的数据库平台是h2
spring.h2.console.settings.web-allow-others=true  # 进行该配置后，h2 web consloe就可以在远程访问了。否则只能在本机访问。
spring.h2.console.path=/h2  #进行该配置，你就可以通过YOUR_URL/h2访问h2 web consloe。YOUR_URL是你程序的访问URl。
spring.h2.console.enabled=true  #进行该配置，程序开启时就会启动h2 web consloe。当然这是默认的，如果你不想在启动程序时启动h2 web consloe，那么就设置为false。
```