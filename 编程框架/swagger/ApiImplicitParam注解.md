# ApiImplicitParams、ApiImplicitParam

swagger 的注解，作用在方法上，表示单独的请求参数 。

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiImplicitParam {
    String name() default ""; //参数名

    String value() default ""; //参数的具体意义，作用

    String defaultValue() default "";

    String allowableValues() default "";

    boolean required() default false; //参数是否必须

    String access() default "";

    boolean allowMultiple() default false;

    String dataType() default ""; //参数的具体类型

    Class<?> dataTypeClass() default Void.class;

    String paramType() default ""; //查询参数类型，这里有几种形式,看下面的描述

    String example() default "";

    Example examples() default @Example({@ExampleProperty(
    mediaType = "",
    value = ""
)});

    String type() default "";

    String format() default "";

    boolean allowEmptyValue() default false;

    boolean readOnly() default false;

    String collectionFormat() default "";
} 

```

类型   作用
path   以地址的形式提交数据
query   直接跟参数完成自动映射赋值
body   以流的形式提交 仅支持POST
header   参数在request headers 里边提交
form   以form表单的形式提交 仅支持POST

在这里我被坑过一次：当我发POST请求的时候，当时接受的整个参数，不论我用body还是query，后台都会报Body Missing错误。这个参数和SpringMvc中的@RequestBody冲突，索性我就去掉了paramType，对接口测试并没有影响。

@ApiImplicitParams：
用于方法，包含多个 @ApiImplicitParam： 
例：

```java
 @ApiOperation("查询测试")
 @GetMapping("select")
 //@ApiImplicitParam(name="name",value="用户名",dataType="String", paramType = "query")
 @ApiImplicitParams({
 @ApiImplicitParam(name="name",value="用户名",dataType="string", paramType = "query",example="xingguo"),
 @ApiImplicitParam(name="id",value="用户id",dataType="long", paramType = "query")})
```



