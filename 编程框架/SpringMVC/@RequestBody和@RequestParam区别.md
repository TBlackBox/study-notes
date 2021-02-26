# 简介
@RequestParam和@RequestBody容易搞混淆，下面简单说一哈区别和用法。
## @RequestParam
注解@RequestParam接收的参数是来自HTTP请求体或请求url的QueryString中。

RequestParam可以接受简单类型的属性，也可以接受对象类型。

### @RequestParam有三个配置参数：

- required 表示是否必须，默认为 true，必须。
- defaultValue 可设置请求参数的默认值。
- value 为接收url的参数名（相当于key值）。


@RequestParam用来处理 Content-Type 为 application/x-www-form-urlencoded 编码的内容，Content-Type默认为该属性。@RequestParam也可用于其它类型的请求，例如：POST、DELETE等请求。

所以在postman中，要选择body的类型为 x-www-form-urlencoded，这样在headers中就自动变为了 Content-Type : application/x-www-form-urlencoded 编码格式。


## @RequestBody

注解@RequestBody接收的参数是来自requestBody中，即请求体。一般用于处理非 Content-Type: application/x-www-form-urlencoded编码格式的数据，比如：application/json、application/xml等类型的数据。

`application/json`类型的数据而言，使用注解@RequestBody可以将body里面所有的json数据传到后端，后端再进行解析。

### 不适用get请求
GET请求中，因为没有HttpEntity，所以@RequestBody并不适用。

POST请求中，通过HttpEntity传递的参数，必须要在请求头中声明数据的类型Content-Type

## SpringMVC的转换
SpringMVC通过使用HandlerAdapter 配置的HttpMessageConverters来解析HttpEntity中的数据，然后绑定到相应的bean上。

### @RequestBody批量接受数据
解析json数据,controller写法
```java
@PostMapping
public void test(@RequestBody List<User> users) {
}
```
由于@RequestBody可用来处理 Content-Type 为 application/json 编码的内容。所以在postman中，选择body的类型为row -> JSON(application/json)，这样在 Headers 中也会自动变为 Content-Type : application/json 编码格式。
发送的数据：

```json
[
    {
        username:"dd",
        password:"22"
    },
    {
        username:"dd",
        password:"22"
	}
]

```
body 里面的 json 语句的 key 值要与后端实体类的属性一一对应。

注意：前端使用$.ajax的话，一定要指定 contentType: "application/json;charset=utf-8;"，默认为 application/x-www-form-urlencoded。

### 通过Map 接受json数据
上面实体类中的具体写法，那么如果传递到非实体类中，body里面的json数据需要怎么解析呢？在controller通过map来接受
```java
@PostMapping
public void testMap(@RequestBody List<Map<String, String>> maps) {
}
```
发送数据跟上面可以一样，结构的话想一哈都明白了，都不叙述了。


# 总结

## **1、从content-type方面总结：**
- form-data、x-www-form-urlencoded：不可以用@RequestBody；可以用@RequestParam。
- application/json：json字符串部分可以用@RequestBody；url中的?后面参数可以用@RequestParam。

 

## **2、从两种注解方式总结：**

**@RequestBody**

```
(@RequestBody Map map)
(@RequestBody Object object)
application/json时候可用
form-data、x-www-form-urlencoded时候不可用
```

**@RequestParam**

```
(@RequestParam Map map)
application/json时候，json字符串部分不可用，url中的?后面添加参数即可用，form-data、x-www-form-urlencoded时候可用，但是要将Headers里的Content-Type删掉
(@RequestParam String waterEleId,@RequestParam String enterpriseName)

application/json时候，json字符串部分不可用，url中的?后面添加参数即可用

form-data、x-www-form-urlencoded时候可用，且参数可以没有顺序（即前端传过来的参数或者url中的参数顺序不必和后台接口中的参数顺序一致，只要字段名相同就可以），但是要将Headers里的Content-Type删掉
(@RequestParam Object object)
不管application/json、form-data、x-www-form-urlencoded都不可用
```

**既不是@RequestBody也不是@RequestParam，没有指定参数哪种接收方式**

```
(Map map)

(Object object)
application/json时候：json字符串部分不可用，url中的?后面添加参数不可用。
因为没有指定，它也不知道到底是用json字符串部分还是?后面添加参数部分，所以干脆都不可以用
form-data、x-www-form-urlencoded时都不可用，见图二
(HttpServletRequest request)
application/json不可用
form-data、x-www-form-urlencoded时可用
```

# **GET请求**

**@RequestBody**

```
RequestBody -- Map / Object
GET请求中不可以使用@RequestBody
```

**@RequestParam**

```
(@RequestParam Map map)

在url中的?后面添加参数即可使用
(@RequestParam String waterEleId,@RequestParam String enterpriseName)
在url中的?后面添加参数即可使用
(@RequestParam Object object)

GET请求中不可以使用
```

当使用GET请求时，通过postman添加?后面的参数，不用在url中自己一个一个拼，点击Params，在下面key-value中输入就自动拼接到url中