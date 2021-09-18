# 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

# 配置
```properties
spring:
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/customer?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2b8&allowMultiQueries=true&useAffectedRows=true
    username: root
    password: 123456
    
  jpa:
      hibernate:
         dialect: org.hibernate.dialect.MySQLInnoDBDialect
         new_generator_mappings: false
         format_sql: true
         ddl_auto: update
         show_sql: true
```