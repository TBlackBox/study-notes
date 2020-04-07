# Gradle常用命令
Gradle版本：2.4

+ 执行特定的task
```
gradle [taskName]
```
+ 清除
```
gradle clean
```
+ 构建
```
gradle build
```
+ 跳过测试构建
```
gradle build -x test
```
+ 显示task之间的依赖关系
```
gradle tasks --all
```
+ 查看testCompile的依赖情形
```
gradle -q dependencies --configuration testCompile
```
+ 继续执行task而忽略前面失败的task
```
gradle build --continue
```
+ 使用指定的gradle文件调用task
```
gradle -b [file_path] [task]
```
+ 使用指定的目录调用task
```
gradle -q -p [dir] helloWorld
```
在指定目录搜索settings.gradle和build.gradle文件。

+ 产生build运行时间的报告
```
gradle build --profile
```
结果存储在build/report/profile目录，名称为build运行的时间。

+ 试运行build
```
gradle -m build
```
+ Gradle的图形界面
```
gradle --gui
```

此外，Gradle的命令日志输出有ERROR（错误信息）、QUIET（重要信息）、WARNGING（警告信息）、LIFECYCLE（进程信息）、INFO（一般信息）、DEBUG（调试信息）一共六个级别。在执行Gradle task时可以适时的调整信息输出等级，以便方便地观看执行结果：

* -q/--quiet启用重要信息级别，该级别下只会输出自己在命令行下打印的信息及错误信息。
* -i/--info会输出除debug以外的所有信息。
* -d/--debug会输出所有日志信息。
* -s/--stacktrace会输出详细的错误堆栈。