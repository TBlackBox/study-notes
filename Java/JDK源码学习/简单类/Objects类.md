# 简介

在JDK7添加了一个Objects工具类(在java.util包下)，它提供了一些方法来操作对象，它由一些静态的实用方法组成，这些方法是null-save（空指针安全的）或null-tolerant（容忍空指针的），用于计算对象的hashcode、返回对象的字符串表示形式、比较两个对象。在比较两个对象的时候，Object的equals方法容易抛出空指针异常，而Objects类中的equals方法就优化了这个问题。



| 修饰和返回类型 | 方法和描述                                       |
| :------------------- | :----------------------------------------------------------- |
| `static  int`     | `compare(T a, T b, Comparator c)`返回0，如果参数都是相同的， `c.compare(a, b)`其他。 |
| `static boolean`  | `deepEquals(Object a, Object b)`返回 `true`如果参数是深层相等，彼此 `false`其他。 |
| `static boolean`  | `equals(Object a, Object b)`返回 `true`如果参数相等，彼此 `false`其他。 |
| `static int`      | `hash(Object... values)`为输入值序列生成哈希码。             |
| `static int`      | `hashCode(Object o)`返回非的哈希码 `null`参数，0为 `null`参数。 |
| `static boolean`  | `isNull(Object obj)`返回 `true`如果提供的引用是 `null`否则返回 `false` 。 |
| `static boolean`  | `nonNull(Object obj)`退货 `true`如果提供的参考是非 `null`否则返回 `false` 。 |
| `static  T`       | `requireNonNull(T obj)`检查指定的对象引用不是 `null` 。      |
| `static  T`       | `requireNonNull(T obj, String message)`检查指定的对象引用不是`null`并抛出自定义的`NullPointerException`如果是）。 |
| `static  T`       | `requireNonNull(T obj, Supplier messageSupplier)`检查指定的对象引用不是`null`并抛出一个自定义的`NullPointerException`（如果是）。 |
| `static String`   | `toString(Object o)`返回非 `null`参数调用 `toString`和 `"null"`参数的 `"null"`的 `null` 。 |
| `static String`   | `toString(Object o, String nullDefault)`如果第一个参数不是 `null` ，则返回第一个参数调用 `toString`的结果， `toString`返回第二个参数。 |



