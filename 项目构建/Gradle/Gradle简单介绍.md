# 简介
Gradle是目前正在开始流行的新一代构建工具，正在逐步的被大家推广使用，尤其以Android为典型。基本上现在所有的Android项目都采用Gradle做为项目构建工具（介绍Gradle版本：2.4）。


# 特点

1. 采用了Groovy DSL来定义配置，相比起XML更加易于学习和使用，并大大简化了构建代码的行数。此外，这种“配置即代码”的方式能够大大简化配置学习和插件编写的成本，提供了更好的灵活性。
2. 在构建模型上非常灵活。可以轻松创建一个task，并随时通过depends语法建立与已有task的依赖关系。这里Gradle使用了Java插件来支持Java项目的标准构建生命周期（和Maven类似）。
3. 依赖的scope简化为compile、runtime、testCompile、testRuntime四种。
4. 支持动态的版本依赖。在版本号后面使用+号的方式可以实现动态的版本管理。
5. 支持排除传递性依赖或者干脆关闭传递性依赖。
6. 完全支持Maven、Ivy的资源库（Repository）。


# 例子
Gradle的配置写在build.gradle文件中。
```
apply plugin: 'groovy'
apply plugin: 'idea'
apply plugin: 'checkstyle'

// --- properties ---
ext.ideaInstallationPath = '/Applications/IntelliJ IDEA.app/Contents'
ext.pomCfg = {
  name 'me.rowkey.libs'
  description '<project_desc>'
  ...
}
sourceCompatibility = 1.6
// --- properties ---

//源码目录结构
sourceSets.main.java.srcDirs = []
sourceSets.main.groovy.srcDir 'src/main/java'

//增加repository
repositories {
    mavenLocal()
    maven{
        url "http://maven.etouch.cn/nexus/content/groups/public/"
    }
    mavenCentral()
}

//依赖管理
dependencies {
    compile fileTree(dir: ideaInstallationPath + '/lib', include: '*.jar')
    testCompile 'org.mockito:mockito-core:2.0.3-beta'
    testCompile 'org.assertj:assertj-core:1.7.1'
    testCompile 'org.springframework:spring-test:4.0.0.RELEASE'
    
    // 排除全部或特定的间接依赖
    runtime ('commons-dbcp:commons-dbcp:1.4') {
        transitive = false
        // 或 exclude group: xxx, module: xxx
    }

    // 然后显式声明
    runtime 'commons-pool:commons-pool:1.6'
}

//for gradle wrapper
task wrapper(type: Wrapper) {
    gradleVersion = '3.0'
}

task helloWorld

helloWorld << {
    println "Hello World!"
}

task testA(dependsOn:helloWorld)

testA << {
    println "test"
}
task copyFile(type: Copy)
//task(copyFile(type: Copy))

copyFile {
    from 'xml'
    into 'destination'
}

//发布到Maven库的配置
uploadArchives {
  repositories {
    mavenDeployer {
      repository(url: <repo_url>) {
       //身份认证信息推荐放在$HOME/.gradle/gradle.properties中
        authentication(
          userName: <repo_user>,
          password: <repo_passwd>)
      }
      pom.project pomCfg
    }
  }
}

```
可见，依赖的配置（dependencies）相比maven,得到了大大的简化，对于任务的定义（task）也非常简单。

# 多模块
首先，在工程的根目录下，创建settings.gradle。
```
 include "common", "api"
 ```
以上即表示包含根目录下的两个子模块common和api。

在根目录的build.gradle中定义公共构建逻辑：
```
 subprojects {
 	apply plugin: 'java'

 	repositories {
 		mavenCentral()
 	}

 	// 所有的子模块共同的依赖
 	dependencies {
 		...
 	}
 }
 ```
subprojects中定义的内容对所有子模块都有效，包括属性、依赖以及Task定义。

需要注意的是在多模块配置下，gradle命令会对所有子模块都执行。如果想要针对单个模块，需要指定模块前缀，如:
```
 gradle :common:clean
 ```

在子模块下创建build.gradle，其中的配置可以增量覆盖父工程中的公共配置。如:
```
 ...

 dependencies {
 	compile project(':common')
 	...
 }
 ```
以上声明了此模块依赖于common模块，当构建此模块时会首先编译打包common模块。相比起Maven每次都要install所依赖的模块，大大简化了使用。

# 注意
1. gradle -Penv=test可以传递参数，使用env引用即可。这里需要注意的是Gradle中默认并没有提供Maven的profile支持，但是可以利用-P参数自己实现此功能。
2. Gradle中的配置中的语法和平常所见的Groovy非常不同, 其利用了Groovy的AST转换等特性实现了自己的一套语法。
3. 建议使用gradle wrapper（gradle wrapper --gradle-version 3.0）来做gradle操作（./gradlew clean）。一方面可以使得项目成员不用预先安装好Gradle，还能够统一项目所使用的Gradle版本。
4. 务必要保持构建脚本简洁、清晰，如：把属性、常量（如版本号）放到gradle.properties中。
5. 模块化构建脚本，通过plugin机制在多个项目中重用，共享相关的配置：apply from: <link_to_gradle>。通过此种方式可以统一团队或者公司的一些构建规范、依赖版本等。