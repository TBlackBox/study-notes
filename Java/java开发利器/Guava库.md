# 简介
Guava是Goole开源的一个涵盖了字符串处理，缓存，并发库，事件总线，i/o等常用的java核心库，也是Goole自己很多Java项目依赖的工具库。

# 文档地址
[githb地址](https://github.com/google/guava)

# 引入
1. maven 引入

+ The JRE flavor requires JDK 1.8 or higher.
+ If you need support for JDK 1.7 or Android, use the Android flavor. You can find the Android Guava source in the android directory.
```
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>29.0-jre</version>
  <!-- or, for Android: -->
  <version>29.0-android</version>
</dependency>
```

2. grade 引入
```
dependencies {
  // Pick one:

  // 1. Use Guava in your implementation only:
  implementation("com.google.guava:guava:29.0-jre")

  // 2. Use Guava types in your public API:
  api("com.google.guava:guava:29.0-jre")

  // 3. Android - Use Guava in your implementation only:
  implementation("com.google.guava:guava:29.0-android")

  // 4. Android - Use Guava types in your public API:
  api("com.google.guava:guava:29.0-android")
}
```