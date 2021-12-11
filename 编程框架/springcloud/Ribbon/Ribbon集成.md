# Ribbon集成

## eureka的新版本，默认集成了ribbon,所以使用了eureka新版本，就可以不用集成ribbon

##  手动引入ribbon

1. 在服务调用方，引入java包

```xml
<dependency>
    <groupId>com.netflix.ribbon</groupId>
    <artifactId>ribbon</artifactId>
    <version>2.2.2</version>
</dependency>
```

### 注意

RestTemplate的：

- xxxForobject()方法，返回的是响应体的数据（可以理解为json数据）

- xxxForEntity()方法，返回的是entity对象，这个对象不仅仅包含响应体数据，还包含响应体信息（状态码等）