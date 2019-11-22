# 简介
json常用的数据结构

# 能表示的几种类型
```
number：和JavaScript的number完全一致；
boolean：就是JavaScript的true或false；
string：就是JavaScript的string；
null：就是JavaScript的null；
array：就是JavaScript的Array表示方式——[]；
object：就是JavaScript的{ ... }表示方式。
```

内部知识：

JSON还定死了字符集必须是UTF-8，表示多语言就没有问题了。为了统一解析，JSON的字符串规定必须用双引号""，Object的键也必须用双引号""。

由于JSON非常简单，很快就风靡Web世界，并且成为ECMA标准。几乎所有编程语言都有解析JSON的库，而在JavaScript中，我们可以直接使用JSON，因为JavaScript内置了JSON的解析。

把任何JavaScript对象变成JSON，就是把这个对象序列化成一个JSON格式的字符串，这样才能够通过网络传递给其他计算机。

如果我们收到一个JSON格式的字符串，只需要把它反序列化成一个JavaScript对象，就可以在JavaScript中直接使用这个对象了。

# 序列化

1. 序列化为字符串
```
var s = JSON.stringify(xiaoming);
```

注意:`JSON.stringify()`这个函数可以传3个参数，如`JSON.stringify(json对象, null, '  ');`

第二个参数可以是数组：表示只需要序列对象中那些属性`JSON.stringify(json对象, ['属性1', '属性2'], '  ');`.也可以是函数
```
function convert(key, value) {
    if (typeof value === 'string') {
        return value.toUpperCase();
    }
    return value;
}

JSON.stringify(xiaoming, convert, '  ');
```
这样对象的每个键值对都会被函数先处理


# 反序列化
+ `JSON.parse()`json字符串转对象
1. 第一种转法
```
JSON.parse('{"name":"小明","age":14}'); // Object {name: '小明', age: 14}
```

2. 第二种转法
`JSON.parse()`还可以接收一个函数，用来转换解析出的属性
```
var obj = JSON.parse('{"name":"小明","age":14}', function (key, value) {
    if (key === 'name') {
        return value + '同学';
    }
    return value;
});
console.log(JSON.stringify(obj)); // {name: '小明同学', age: 14}
```