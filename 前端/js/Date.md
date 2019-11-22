# 简介
在JavaScript中，Date对象用来表示日期和时间。

要获取系统当前时间。

```
var now = new Date();
now; // Wed Jun 24 2015 19:49:22 GMT+0800 (CST)
now.getFullYear(); // 2015, 年份
now.getMonth(); // 5, 月份，注意月份范围是0~11，5表示六月
now.getDate(); // 24, 表示24号
now.getDay(); // 3, 表示星期三
now.getHours(); // 19, 24小时制
now.getMinutes(); // 49, 分钟
now.getSeconds(); // 22, 秒
now.getMilliseconds(); // 875, 毫秒数
now.getTime(); // 1435146562875, 以number形式表示的时间戳
```

注意：这个是从本机中获取，不准确，系统时间是可以改的。


# 构建日期对象

1. 一种构建
```
var d = new Date(2015, 5, 19, 20, 15, 30, 123);
d：是时间搓
```
注意：月份是从0开始哦 ，意思计算 0表示1月  11 表示12月，设计者唠嗑有包。

2. 二种构建
```
var d = Date.parse('2015-06-24T19:49:22.875+08:00');
```
注意：参数要符合符合`ISO 8601`格式的字符串哦,返回时时间搓。时间搓可以这样转日期
```
var d = new Date(1435146562875);
d; // Wed Jun 24 2015 19:49:22 GMT+0800 (CST)
d.getMonth(); // 5
```

3. 显示时区
Date对象表示的时间总是按浏览器所在时区显示的，不过我们既可以显示本地时间，也可以显示调整后的UTC时间
```
var d = new Date(1435146562875);
d.toLocaleString(); // '2015/6/24 下午7:49:22'，本地时间（北京时区+8:00），显示的字符串与操作系统设定的格式有关
d.toUTCString(); // 'Wed, 24 Jun 2015 11:49:22 GMT'，UTC时间，与本地时间相差8小时
```