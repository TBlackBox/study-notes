# 简介
Arrays 是在`java.util`包下的一个工具类，里面定义了对数组的一些操作方法。
- 包含用来操作数组（比如排序和搜索）的各种方法。
- 包含一个允许将数组作为列表来查看的静态工厂。 

***除非特别注明，否则如果指定数组引用为 null，则此类中的方法都会抛出 NullPointerException***

# 常用方法介绍
## asList()
返回一个受指定数组支持的固定大小的列表。（对返回列表的更改会“直接写”到数组。）此方法同 Collection.toArray() 一起，充当了基于数组的 API 与基于 collection 的 API 之间的桥梁。返回的列表是可序列化的，并且实现了 RandomAccess。
### 方法原型
```java
public static <T> List<T> asList(T... a) {
	return new ArrayList<>(a);
}
```
### 使用方法
```java
List<Integer> list = Arrays.asList(1,2,3,4);
```
*** 注意：Arrays.asList() 方法返回的是一个固定大小的数组 , 也就是说不可以进行**add **和 remove 操作***


## toString
返回指定数组内容的字符串表示形式。字符串表示形式由数组的元素列表组成，括在方括号（"[]"）中。相邻元素用字符 ", "（逗号加空格）分隔。这些元素通过 String.valueOf(double) 转换为字符串。如果 a 为 null，则返回 "null"。
### 原型
```java
//当人  还有很多其他类型
public static String toString(int[] a) {}
```
### 列子
```java
int[] array = {32,43,21,23,12};
//输出： [32, 21, 23, 43, 12]
System.out.println(Arrays.toString(array));
```
## sort 数组排序

### 函数原型
```java
// 对数组内所有元素进行排序.
public static void sort(char[] a);
// 对指定范围内的数据进行排序. 包括fromIndex 不包括 toIndex
public static void sort(char[] a,int fromIndex,int toIndex);
// 其他的类型的...   例如int  long 等等
```
### 案列
```java
@Test
	public void testSort() {
		Integer[] src = {2,1,2,2,32,12,12};
		Arrays.sort(src);
		
		
		//对整个数组排序 输出：[1, 2, 2, 2, 12, 12, 32]
		System.out.println(Arrays.toString(src));
		
		int[] array = {32,43,21,23,12};
		Arrays.sort(array,1,4);
		//对指定索引下排序  输出： [32, 21, 23, 43, 12]
		System.out.println(Arrays.toString(array));
		
		int[] pallArr = {12,221,112,11,21,212,321,312,12,12,12,231,1212,31};
		Arrays.parallelSort(pallArr);
		//从小到大的排序 输出：[11, 12, 12, 12, 12, 21, 31, 112, 212, 221, 231, 312, 321, 1212]
		System.out.println(Arrays.toString(pallArr));
		
		Integer[] comparatorArr = {23,12,32,321,1234,21,23,42,12,3,4};
		

		//上面的都是自然顺序  如果要大到小排列 可以这样写，这里使用到了策略模式，这样非常方便
		//传统写法
//		Arrays.sort(comparatorArr, new DifineComparator());
		//表达式的写法
		Arrays.sort(comparatorArr, (o1,o2) ->{
//			返回值为正值，把o1排在o2后面；
//			返回值为负值，把o1排在o2前面。
//			如果返回值是0，按照容器之前的顺序排列。
			return o2 -o1;
		});
		//输出 [1234, 321, 42, 32, 23, 23, 21, 12, 12, 4, 3]
		System.out.println(Arrays.toString(comparatorArr));
	}
	
	class DifineComparator implements Comparator<Integer>{

		@Override
		public int compare(Integer o1, Integer o2) {
			return o1 - o2;
		}
		
	}
```
 注意： `parallelSort()`是jdk1.8新增的，基于工作窃取的方式排序，在处理大量的排序数据的情况下，可能又会大的性能提升。

 ### 引用类型排序
```java
public class Test {
    public static void main(String[] args) {
        Man[] msMans = {new Man(3, "a"), new Man(60, "b"),new Man(2, "c")};
        Arrays.sort(msMans);
        System.out.println(Arrays.toString(msMans));
    }
}
 
class Man implements Comparable {
    int age;
    int id;
    String name;
 
    public Man(int age, String name) {
        super();
        this.age = age;
        this.name = name;
    }
 
    public String toString() {
        return this.name;
    }
 
    public int compareTo(Object o) {
        Man man = (Man) o;
        if (this.age < man.age) {
            return -1;
        }
        if (this.age > man.age) {
            return 1;
        }
        return 0;
    }
}
```
反正值说明：
对于参与比较的两个Object，o1和o2，如果函数的
返回值为正值，把o1排在o2后面；
返回值为负值，把o1排在o2前面。
如果返回值是0，按照容器之前的顺序排列。
***  注意：***
在compareTo中，this相当于o1,传入的Object相当于o2
### 总结
Java中 实现自定义排序有两种方式，一种技术继承`Comparable`接口，实现`compareTo()`方法，一种就是使用`Comparator`接口。

## binarySearch 二分查找
返回元素在数组中出现的索引，如果元素不存在，则返回的是（-插入点-1）,也就是说它会根据你需要查找的元素应该存在的位置索引，将该索引值再减去1即可。

### 原型
这里举例一种，当然还有很多其他类型。
```java
 public static int binarySearch(int[] a, int key) {}
```
### 案列
```java
int[] a = {122,222,12,221,2,13,75654,321,128,12};

//将数值进行升序排序(这里必须进行升序排序)
Arrays.sort(a);
//[2, 12, 12, 13, 122, 128, 221, 222, 321, 75654]
// 0   1  2    3   4    5   6    7    8    9
System.out.println(Arrays.toString(a));

int index = Arrays.binarySearch(a, 321);
System.out.println(index); //8
int xindex = Arrays.binarySearch(a, 50);
System.out.println(xindex); //-4
int yindex = Arrays.binarySearch(a, 12);
System.out.println(yindex); //1
```

## copyOf 数组复制
复制指定的数组，截取或用 0 填充（如有必要），以使副本具有指定的长度。对于在原数组和副本中都有效的所有索引，这两个数组将包含相同的值。对于在副本中有效而在原数组无效的所有索引，副本将包含 (byte)0。当且仅当指定长度大于原数组的长度时，这些索引存在。

### 原型
这里只列举了int 类型   当然还有其他类型。
```java
public static int[] copyOf(int[] original, int newLength) {
        int[] copy = new int[newLength];
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
}
```
### 案列
```java
int[] src = {211,21,321};

int[] dest = Arrays.copyOf(src, src.length + 1);
//输出 [211, 21, 321, 0]
System.out.println(Arrays.toString(dest));

int[] dest2 = Arrays.copyOf(src, src.length - 1);
//输出 [211, 21]
System.out.println(Arrays.toString(dest2));

int[] dest3 = Arrays.copyOfRange(src, 1,2);
//输出 [21]
System.out.println(Arrays.toString(dest3));
```

### copyOfRange 复制数组指定范围.
将指定数组的指定范围复制到一个新数组。该范围的初始索引 (from) 必须位于 0 和 original.length（包括）之间。original[from] 处的值放入副本的初始元素中（除非 from == original.length 或 from == to）。原数组中后续元素的值放入副本的后续元素。该范围的最后索引 (to) （必须大于等于 from）可以大于 original.length，在这种情况下，false 被放入索引大于等于 original.length - from 的副本的所有元素中。返回数组的长度为 to - from。
### 原型
```java
public static int[] copyOfRange(int[] original, int from, int to)
```

## equals 数组比较
如果两个指定的 boolean 型数组彼此相等，则返回 true。如果两个数组包含相同数量的元素，并且两个数组中的所有相应元素对都是相等的，则认为这两个数组是相等的。换句话说，如果两个数组以相同顺序包含相同的元素，则两个数组是相等的。此外，如果两个数组引用都为 null，则认为它们是相等的。

```java
public static boolean equals(boolean[] a,boolean[] a2)
```

## fill 数组填充
将指定的 boolean 值分配给指定 boolean 型数组指定范围中的每个元素。填充的范围从索引 fromIndex（包括）一直到索引 toIndex（不包括）。（如果 fromIndex==toIndex，则填充范围为空。）
### 原型
这里指的是boolean,当然还有其他类型。
```
// 用指定的数据填充数组.
public static void fill(boolean[] a,boolean val)
// 用指定的数据填充指定的区间
public static void fill(boolean[] a,int fromIndex,int toIndex, boolean val);
```