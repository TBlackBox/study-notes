# Period
Period在概念上和Duration类似，区别在于Period是以年月日来衡量一个时间段，比如4年5个月23天

## 通过of()创建
```java
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
