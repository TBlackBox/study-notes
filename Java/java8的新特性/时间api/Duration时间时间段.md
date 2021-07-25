
# Duration
## Duration与Instant比较
相同点：  
Duration的内部实现与Instant类似，也是包含两部分：seconds表示秒，nanos表示纳秒。
区别：  
Instant用于表示一个时间戳（或者说是一个时间点），而Duration表示一个时间段，所以Duration类中不包含now()静态方法。

## 通过 Duration.between()方法创建Duration对象
```java
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
```java
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