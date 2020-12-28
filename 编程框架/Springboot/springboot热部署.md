# 使用springboot推荐的devtools
1. 在pom文件引入依赖
```
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<optional>true</optional>
</dependency>

```
2. 在idea里面编写完代码后ctrl+f9就可以重新编译了
即是：build---->build project

3. eclipse里面通过ctrl+s保存即可


# JRebel
+ 收费的一个热部署软件
+ 安装插件即可用

缺点： 
 收费


# Spring Loaded热部署
 这里简单介绍一下，Spring官方提供的热部署程序，实现修改类文件的热部署
 + [项目下载地址](https://github.com/spring-projects/spring-loaded)
 + 添加运行时参数
 -javaagent:C:/springloaded-1.2.5.RELEASE.jar -noverify
指定程序位置
缺点：用起比较麻烦





