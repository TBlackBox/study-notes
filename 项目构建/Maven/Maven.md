
maven install 跳过测试
mvn命令跳过测试：
mvn install -Dmaven.test.skip=true 测试类不会生成.class 文件
mvn install -DskipTests 测试类会生成.class文件

使用maven-surefire-plugin跳过测试需要在该插件配置下加：
```
<configuration> 
<skipTests>true</skipTests> 
</configuration>

```

测试类会生成.class文件

spring boot项目跳过测试，使用spring-boot-maven-plugin，需要在pom.xml里加：
```
<properties>
<skipTests>true</skipTests>
</properties>
```
测试类会生成.class文件