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


#  LocalDataTime 转秒和毫秒
```
//获取秒数
Long second = LocalDateTime.now().toEpochSecond(ZoneOffset.of("+8"));
//获取毫秒数
Long milliSecond = LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli();
```







