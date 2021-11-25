# Maven常用命令（maven版本3.3.9）

+ 清除
```
mvn -N clean
```
-N参数表示仅仅构建当前目录的项目，不去构建子模块。

+ 打包
```
mvn package
```
+ 发布到本地
```
mvn install
```
+ 发布到线上
```
mvn deploy
```
+ 将依赖复制到指定目录
```
mvn dependency:copy-dependencies -DoutputDirectory=./lib
```

+ 部署非Maven项目的jar包
```
mvn deploy:deploy-file -DgroupId=[groupId] -DartifactId=[artifactId] -Dversion=[version] -Dpackaging=jar -Dfile=[jarFilePath] -Durl=[repositoryUrl]
```
+ 安装非Maven项目的jar包到本地
```
mvn install:install-file -DgroupId=[groupId] -DartifactId=[artifactId] -Dversion=[version] -Dpackaging=jar -Dfile=[jarFilePath]
```
+ 执行指定类中的main方法
```
mvn exec:java -Dexec.mainClass=[mainClass]
```
+ 查看依赖树
```
mvn dependency:tree
```
+ 执行指定的测试用例
```
mvn test -Dtest=[ClassName]#[MethodName] #[MethodName]为要运行的方法名，支持*通配符
```
+ 跳过测试阶段且不编译测试用例类
```
mvn -Dmaven.test.skip=true ...
```
+ 跳过测试阶段但编译测试用例类
```
mvn -DskipTests ...
```
+ 使用指定的pom文件或者指定目录下的pom.xml运行
```
mvn -f [file/dir] ...
```
+ 从已有项目生成archetype
```
mvn archetype:create-from-project
```
+ 从Maven库搜索archetype
```
mvn archetype:crawl
```
+ 根据archetype生成项目
```
mvn archetype:generate -DgroupId=[groupId] -DartifactId=[artifactId] -DarchetypeArtifactId=[archetypeArtifactId] -DarchetypeVersion=[archetypeVersion] -DarchetypeCatalog=[archetypeCatalogPath]
```
需要注意，3.x版本去掉了archetypeRepository参数并且修改了archetypeCatalog参数，需要在settings.xml中配置repository。如果需要在参数中设置，可以使用mvn 
org.apache.maven.plugins:maven-archetype-plugin:2.4:generate配合-DarchetypeRepository=http://xxx[远程repository的url]或者-DarchetypeCatalog=http://xxx[catalog的远程url]来使用远程的archetype。

此外，可以使用-q参数使Maven的日志输出只包含错误信息。



- 循环删除`lastUpdated` 文件

  ```
  for /r %i in (*.lastUpdated)do del %i
  ```

  