# 简介
在jdk1.8之前，对时间得格式操作比较麻烦，并且对格式的转换不是线程安全的，在jdk8中，加入了全套时间api,方便得格式转换，在`java.time`包下，分别定义了时间相关的api,下面分别简单介绍一哈。

# LocalDate,LocalTime,LocalDateTime
这几个类的实例是不可变的对象，分别表示使用ISO-8601日历系统的日期，时间，日期和时间，他们提供了简单的日期或时间，并包含当前的时间信息。也包含与时区相关的信息。在这里简单描述`LocalDateTime`个就是，其他的差不多,这几个类是给人看的，人可以识别的。

## LocalDateTime的使用
1. 获取当前时间创建对象
```
//通过now()获取
LocalDateTime localDateTime = LocalDateTime.now();
//通过of()获取  输出     2020-01-13T12:41:00.007
System.out.println(localDateTime);

//of(参数) 有多个静态方法，可以根据自己的需要选择
LocalDateTime localDateTime2 = LocalDateTime.of(2020, 1, 21, 2, 21);
//输出   2020-01-21T02:21
System.out.println(localDateTime2);
```

2. 对时间得增，减，更改操作
```
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println("当前时间："+localDateTime);

/**
* 添加时间
*/
//注意  这里需要一个新变量来接收  LocalDateTime 是不可变的实例
//还有很多方法  这里只演示了添加小时  分  天 秒  毫秒  纳秒  周这些都可以
LocalDateTime localDateTime1 = localDateTime.plusHours(2);
System.out.println("添加2个小时候的时间:"+localDateTime1);

/**
* 减少时间
*/
//在当前时间减少2个月  这里演示的是月  当然还有年 天  秒 分等等  具体看api
LocalDateTime localDateTime2 =  localDateTime.minusMonths(2);
//输出   当前时间减少2个月的时间：2019-11-13T13:04:37.199
System.out.println("当前时间减少2个月的时间：" + localDateTime2);

/**
* 指定时间
*/
//这里指定月中的天  当然  还能指定 年 月  日这些  
LocalDateTime localDateTime3 = localDateTime.withDayOfMonth(2);
//输出 将改月的天指定为2号：2020-01-02T13:10:29.856
System.out.println("将改月的天指定为2号："+localDateTime3);
```

3. 获取时间
```
LocalDateTime localDateTime = LocalDateTime.now();
		System.out.println("时间："+localDateTime);
		
		System.out.println("得到月中的天："+localDateTime.getDayOfMonth());
		System.out.println("得到年中的天："+localDateTime.getDayOfYear());
		System.out.println("得到小时："+localDateTime.getHour());
		System.out.println("得到分钟数："+localDateTime.getMinute());
		System.out.println("得到几月（值）："+localDateTime.getMonthValue());
		System.out.println("得到纳秒："+localDateTime.getNano());
		System.out.println("得到秒："+localDateTime.getSecond());
		System.out.println("得到星期几（英文）："+localDateTime.getDayOfWeek());
		System.out.println("得到月份（英文）："+localDateTime.getMonth());
		//输出
//		时间：2020-01-20T15:02:39.535
//		得到月中的天：20
//		得到年中的天：20
//		得到小时：15
//		得到分钟数：2
//		得到几月（值）：1
//		得到纳秒：535000000
//		得到秒：39
//		得到星期几（英文）：MONDAY
//		得到月份（英文）：JANUARY
```
上面只是一些说明了一些时间api的例子，还有很多，需自己慢慢去看。

4. 时间比较
```
		LocalDateTime localDateTime = LocalDateTime.now();
		System.out.println("时间1：" + localDateTime);
		LocalDateTime localDateTime2 = localDateTime.plusHours(2);
		System.out.println("时间2：" + localDateTime2);
		
		System.out.println("时间2 在时间1之后：" + localDateTime.isAfter(localDateTime2));
		System.out.println("时间2 在时间1之前：" + localDateTime.isBefore(localDateTime2));
//		时间1：2020-01-20T15:13:35.551
//		时间2：2020-01-20T17:13:35.551
//		时间2 在时间1之后：false
//		时间2 在时间1之前：true
		
		//isEqual()  判断时间是否相等 输出 flase
	    System.out.println(localDateTime.isEqual(localDateTime2));
```

## 高级一点的时间调整

比如将时间调到下一个工作日，或者是下个月的最后一天，这时候我们可以使用`with()`方法的另一个重载方法，它接收一个`TemporalAdjuster`参数，可以使我们更加灵活的调整日期：

```
// 返回下一个距离当前时间最近的星期日
LocalDate date7 = date.with(nextOrSame(DayOfWeek.SUNDAY)); 
// 返回本月最后一个星期六
LocalDate date9 = date.with(lastInMonth(DayOfWeek.SATURDAY));  
```

要使上面的代码正确编译，你需要使用静态导入`TemporalAdjusters`对象：

```
import static java.time.temporal.TemporalAdjusters.*;
```

`TemporalAdjusters`类中包含了很多静态方法可以直接使用，下面的表格列出了一些方法：

| 方法名                        | 描述                                                        |
| :---------------------------- | :---------------------------------------------------------- |
| `dayOfWeekInMonth`            | 返回同一个月中每周的第几天                                  |
| `firstDayOfMonth`             | 返回当月的第一天                                            |
| `firstDayOfNextMonth`         | 返回下月的第一天                                            |
| `firstDayOfNextYear`          | 返回下一年的第一天                                          |
| `firstDayOfYear`              | 返回本年的第一天                                            |
| `firstInMonth`                | 返回同一个月中第一个星期几                                  |
| `lastDayOfMonth`              | 返回当月的最后一天                                          |
| `lastDayOfNextMonth`          | 返回下月的最后一天                                          |
| `lastDayOfNextYear`           | 返回下一年的最后一天                                        |
| `lastDayOfYear`               | 返回本年的最后一天                                          |
| `lastInMonth`                 | 返回同一个月中最后一个星期几                                |
| `next / previous`             | 返回后一个/前一个给定的星期几                               |
| `nextOrSame / previousOrSame` | 返回后一个/前一个给定的星期几，如果这个值满足条件，直接返回 |

如果上面表格中列出的方法不能满足你的需求，你还可以创建自定义的`TemporalAdjuster`接口的实现，`TemporalAdjuster`也是一个函数式接口，所以我们可以使用Lambda表达式：

```
@FunctionalInterface
public interface TemporalAdjuster {   
	Temporal adjustInto(Temporal temporal);
}
```

比如给定一个日期，计算该日期的下一个工作日（不包括星期六和星期天）：

```
LocalDate date = LocalDate.of(2017, 1, 5);
date.with(temporal -> {
    // 当前日期
    DayOfWeek dayOfWeek = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK));

    // 正常情况下，每次增加一天
    int dayToAdd = 1;

    // 如果是星期五，增加三天
    if (dayOfWeek == DayOfWeek.FRIDAY) {
        dayToAdd = 3;
    }

    // 如果是星期六，增加两天
    if (dayOfWeek == DayOfWeek.SATURDAY) {
        dayToAdd = 2;
    }

    return temporal.plus(dayToAdd, ChronoUnit.DAYS);
});
```

# Period
Period在概念上和Duration类似，区别在于Period是以年月日来衡量一个时间段，比如4年5个月23天

## 通过of()创建
```
Period period = Period.of(4, 5, 23);
		
int year = period.getYears();
//值：4
System.out.println(year);
Period minPeriod = period.minusYears(2);
//值：2
System.out.println(minPeriod.getYears());
```
还有很多的方法可以用，创建方法也有很多

## 通过between创建
Period对象也可以通过between()方法创建，值得注意的是，由于Period是以年月日衡量时间段，所以between()方法只能接收LocalDate类型的参数：
```
// 2020-01-05 到 2020-02-05 这段时间
Period period = Period.between(
                LocalDate.of(2020, 1, 5),
                LocalDate.of(2020, 2, 5));
```




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

# Duration
## Duration与Instant比较
相同点：  
Duration的内部实现与Instant类似，也是包含两部分：seconds表示秒，nanos表示纳秒。
区别：  
Instant用于表示一个时间戳（或者说是一个时间点），而Duration表示一个时间段，所以Duration类中不包含now()静态方法。

## 通过 Duration.between()方法创建Duration对象
```
//2020-01-10 12:00:00
LocalDateTime from = LocalDateTime.of(2020, Month.JANUARY, 10, 12, 0, 0);   
// 2020-02-10 12:10:10
LocalDateTime to = LocalDateTime.of(2020, Month.FEBRUARY, 10, 12, 10, 10);    
// 表示从2020-01-10 12:00:00 到2020-02-10 12:10:10 这段时间
Duration duration = Duration.between(from, to);    

// 这段时间的总天数    值：31
long days = duration.toDays();
// 这段时间的小时数  值：744
long hours = duration.toHours(); 
// 这段时间的分钟数 值:44650
long minutes = duration.toMinutes(); 
// 这段时间的秒数 值：2679010
long seconds = duration.getSeconds(); 
// 这段时间的毫秒数 值：2679010000
long milliSeconds = duration.toMillis();   
// 这段时间的纳秒数 值：2679010000000000
long nanoSeconds = duration.toNanos(); 
```

## 通过of()创建
```
// 10天  
Duration duration1 = Duration.of(10, ChronoUnit.DAYS);   
// 10000毫秒
Duration duration2 = Duration.of(10000, ChronoUnit.MILLIS);  
```
还有一些静态方法可以进行创建
```
//天为单位
 public static Duration ofDays(long days){};
 //小时为单位
 public static Duration ofHours(long hours) {}
 //分钟为单位
 public static Duration ofMinutes(long minutes) {}
 //等等  还有很多 
```


# 格式化日期
新的日期API中提供了一个DateTimeFormatter类用于处理日期格式化操作，它被包含在java.time.format包中，Java 8的日期类有一个format()方法用于将日期格式化为字符串，该方法接收一个DateTimeFormatter类型参数：
```
LocalDateTime dateTime = LocalDateTime.now();
String strDate1 = dateTime.format(DateTimeFormatter.BASIC_ISO_DATE);    // 20170105
String strDate2 = dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE);    // 2017-01-05
String strDate3 = dateTime.format(DateTimeFormatter.ISO_LOCAL_TIME);    // 14:20:16.998
String strDate4 = dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));   // 2017-01-05
String strDate5 = dateTime.format(DateTimeFormatter.ofPattern("今天是：YYYY年 MMMM DD日 E", Locale.CHINESE)); // 今天是：2017年 一月 05日 星期四

```
同样，日期类也支持将一个字符串解析成一个日期对象，例如：
```
String strDate6 = "2017-01-05";
String strDate7 = "2017-01-05 12:30:05";

LocalDate date = LocalDate.parse(strDate6, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
LocalDateTime dateTime1 = LocalDateTime.parse(strDate7, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

# 时区
Java 8中的时区操作被很大程度上简化了，新的时区类java.time.ZoneId是原有的java.util.TimeZone类的替代品。ZoneId对象可以通过ZoneId.of()方法创建，也可以通过ZoneId.systemDefault()获取系统默认时区：
```
ZoneId shanghaiZoneId = ZoneId.of("Asia/Shanghai");
ZoneId systemZoneId = ZoneId.systemDefault();
```
of()方法接收一个“区域/城市”的字符串作为参数，你可以通过getAvailableZoneIds()方法获取所有合法的“区域/城市”字符串：
```
Set<String> zoneIds = ZoneId.getAvailableZoneIds();
```
对于老的时区类TimeZone，Java 8也提供了转化方法：
```
ZoneId oldToNewZoneId = TimeZone.getDefault().toZoneId();
```
有了ZoneId，我们就可以将一个LocalDate、LocalTime或LocalDateTime对象转化为ZonedDateTime对象：
```
LocalDateTime localDateTime = LocalDateTime.now();
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, shanghaiZoneId);
```
将zonedDateTime打印到控制台为：
```
2017-01-05T15:26:56.147+08:00[Asia/Shanghai]
```
ZonedDateTime对象由两部分构成，LocalDateTime和ZoneId，其中2017-01-05T15:26:56.147部分为LocalDateTime，+08:00[Asia/Shanghai]部分为ZoneId。

另一种表示时区的方式是使用ZoneOffset，它是以当前时间和世界标准时间（UTC）/格林威治时间（GMT）的偏差来计算，例如：
```
ZoneOffset zoneOffset = ZoneOffset.of("+09:00");
LocalDateTime localDateTime = LocalDateTime.now();
OffsetDateTime offsetDateTime = OffsetDateTime.of(localDateTime, zoneOffset);
```

# 其他历法
Java中使用的历法是ISO 8601日历系统，它是世界民用历法，也就是我们所说的公历。平年有365天，闰年是366天。闰年的定义是：非世纪年，能被4整除；世纪年能被400整除。为了计算的一致性，公元1年的前一年被当做公元0年，以此类推。

此外Java 8还提供了4套其他历法（很奇怪为什么没有汉族人使用的农历），每套历法都包含一个日期类，分别是：

- ThaiBuddhistDate：泰国佛教历
- MinguoDate：中华民国历
- JapaneseDate：日本历
- HijrahDate：伊斯兰历
每个日期类都继承ChronoLocalDate类，所以可以在不知道具体历法的情况下也可以操作。不过这些历法一般不常用，除非是有某些特殊需求情况下才会使用。

这些不同的历法也可以用于向公历转换：
```
LocalDate date = LocalDate.now();
JapaneseDate jpDate = JapaneseDate.from(date);
```
由于它们都继承ChronoLocalDate类，所以在不知道具体历法情况下，可以通过ChronoLocalDate类操作日期：
```
Chronology jpChronology = Chronology.ofLocale(Locale.JAPANESE);
ChronoLocalDate jpChronoLocalDate = jpChronology.dateNow();
```
我们在开发过程中应该尽量避免使用ChronoLocalDate，尽量用与历法无关的方式操作时间，因为不同的历法计算日期的方式不一样，比如开发者会在程序中做一些假设，假设一年中有12个月，如果是中国农历中包含了闰月，一年有可能是13个月，但开发者认为是12个月，多出来的一个月属于明年的。再比如假设年份是累加的，过了一年就在原来的年份上加一，但日本天皇在换代之后需要重新纪年，所以过了一年年份可能会从1开始计算。

在实际开发过程中建议使用LocalDate，包括存储、操作、业务规则的解读；除非需要将程序的输入或者输出本地化，这时可以使用ChronoLocalDate类。