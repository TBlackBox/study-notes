# Instant 时间戳
Instant : 时间戳，使用 Unix 元年1970年1月1日 00:00:00 所经历的毫秒值。

## 获取时间
```
//输出 当前时间:2020-01-20T16:00:31.661
System.out.println("当前时间:" + LocalDateTime.now()); 
Instant instant = Instant.now(); //默认使用 UTC 时区
// 输出  instant：2020-01-20T07:58:44.388Z   特别注意 ： 这里的时间和当前时间相差8个小时， 这里用的是UTC 时间
System.out.println("instant：" + instant);

//得到秒的时间搓   输出  1579507124
System.out.println(instant.getEpochSecond());
//得到带毫秒的时间搓  输出  1579507124388
System.out.println(instant.toEpochMilli());
//得到纳秒 输出 388000000
System.out.println(instant.getNano());
```
还有3种获取时间的方式
```
//1970-01-01T00:00:00 + 120毫秒的时间   值： 1970-01-01T00:00:00.120Z
Instant instant1 = Instant.ofEpochMilli(120);
//1970-01-01T00:00:00 + 120秒的时间  值：1970-01-01T00:02:00Z
Instant instant2 = Instant.ofEpochSecond(120);
//1970-01-01T00:00:00 + 120秒 + 1000000纳秒的时间  值：1970-01-01T00:02:00.001Z
Instant instant3 = Instant.ofEpochSecond(120, 1000000);
```