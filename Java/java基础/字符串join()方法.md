# 简介

在jdk 8后 ，提供了join()方法 可以把字符串通过某一字符串连接起来


## 案列
jdk8中，join()重载了两个方法出来，分别是
- 使用可变字符串
```java
 public static String join(CharSequence delimiter, CharSequence... elements) {
        Objects.requireNonNull(delimiter);
        Objects.requireNonNull(elements);
        // Number of elements not likely worth Arrays.stream overhead.
        StringJoiner joiner = new StringJoiner(delimiter);
        for (CharSequence cs: elements) {
            joiner.add(cs);
        }
        return joiner.toString();
    }
```
- 使用数组或者列表
```java
public static String join(CharSequence delimiter,
            Iterable<? extends CharSequence> elements) {
            Objects.requireNonNull(delimiter);
            Objects.requireNonNull(elements);
            StringJoiner joiner = new StringJoiner(delimiter);
            for (CharSequence cs: elements) {
            joiner.add(cs);
            }
            return joiner.toString();
}
```
## 例子
```java
String value = String.join(":", "1","2","3");
//输出：1:2:3
System.out.println(value);

List<String> list = new ArrayList<String>();
list.add("a");
list.add("b");
list.add("c");
String value2 = String.join(":", list);
//输出：a:b:c
System.out.println(value2);

```