# 简介
Integer类是基本数据类型int的包装类，每个Integer都包含了int类型字段，是抽象类Number的子类。
Integer 类被关键字`final`修饰，也就是说不会又子类。

## 属性

```
//MIN_VALUE静态变量表示int能取的最小值，为-2的31次方(-2147483648 )，被final修饰说明不可变。
public static final int   MIN_VALUE = 0x80000000;
//类似的还有MAX_VALUE，表示int最大值为2的31次方减1(2147483647)。
public static final int   MAX_VALUE = 0x7fffffff;
public static final int SIZE = 32;

```
1. 一些不变的常量
*** 注意： ***
- 上面也是Integer和int的取值范围。
- 对于Integer类java为了提高效率，初始化了-128--127之间的整数对象，因此Integer类取值-128--127的时候效率最高。
- 由于补码表示负数的关系，正数总是比负数多一个。
- SIZE用来表示二进制补码形式的int值的比特数，值为32，因为是静态变量所以值不可变。


```
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
```

1. 这个应该是指类型，类型是int。

```
final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                      99999999, 999999999, Integer.MAX_VALUE };
// Requires positive xstatic int stringSize(int x) {
    for (int i=0; ; i++)
        if (x <= sizeTable[i])
            return i+1;
}

```

1. stringSize主要是为了判断一个int型数字对应字符串的长度。sizeTable这个数组存储了该位数的最大值。

```
   final static char[] digits = {
        '0' , '1' , '2' , '3' , '4' , '5' ,
        '6' , '7' , '8' , '9' , 'a' , 'b' ,
        'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
        'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
        'o' , 'p' , 'q' , 'r' , 's' , 't' ,
        'u' , 'v' , 'w' , 'x' , 'y' , 'z'
    };
```

1. digits数组里面存的是数字从二进制到36进制所表示的字符，所以需要有36个字符才能表示所有不用进制的数字。

```
final static char [] DigitTens = {
        '0', '0', '0', '0', '0', '0', '0', '0', '0', '0',
        '1', '1', '1', '1', '1', '1', '1', '1', '1', '1',
        '2', '2', '2', '2', '2', '2', '2', '2', '2', '2',
        '3', '3', '3', '3', '3', '3', '3', '3', '3', '3',
        '4', '4', '4', '4', '4', '4', '4', '4', '4', '4',
        '5', '5', '5', '5', '5', '5', '5', '5', '5', '5',
        '6', '6', '6', '6', '6', '6', '6', '6', '6', '6',
        '7', '7', '7', '7', '7', '7', '7', '7', '7', '7',
        '8', '8', '8', '8', '8', '8', '8', '8', '8', '8',
        '9', '9', '9', '9', '9', '9', '9', '9', '9', '9',
        } ;

final static char [] DigitOnes = {
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        } ;
```

1. DigitTens和DigitOnes两个数组
DigitTens数组是为了获取0到99之间某个数的十位,
DigitOnes是为了获取0到99之间某个数的个位


# Integer和int的区别

1、Integer是int的包装类，int是java的一种基本数据类型 
2、Integer变量必须实例化后才能使用，而int变量不需要 
3、Integer实际是对象的引用，当new一个Integer时，实际上是生成一个指针指向此对象；而int则是直接存储数据值 

非new生成的Integer变量指向的是java常量池中的对象，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同）
```
Integer a = new Integer(12);
Integer b = 12;
System.out.println(a == b); //false
System.out.println(a.equals(b)); //true
```
*** 注意:***
- `Integer.valueOf(12)`这个的效果跟直接赋值的效果是一样的.
- `==`和`equals`
	- == 直接比较的地址,如果Integer的值小于128，两者比较的值是一样的，如果大于等于128，则必须用equals进行比较。

4、Integer的默认值是null，int的默认值是0

Java的Ingeter是int的包装类,在开发中我们基本可以将两者等价。

Integer类是对int进行封装,里面包含处理int类型的方法，比如int到String类型的转换方法或String类型转int类型的方法，也有与其他类型之间的转换方，当然也包括操作位的方法。

# IntegerCache内部类
```
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

IntegerCache是Integer的一个静态内部类，它包含了int可能值得Integer数组，它负责存储了(high -low)个静态Integer对象，并且在静态代码块中初始化。
默认范围是【-128,127】，所以这里默认只实例化了256个Integer对象，当Integer的值范围在【-128,127】时则直接从缓存中获取对应的Integer对象，不必重新实例化。这些缓存值都是静态且final的，避免重复的实例化和回收。所以值得-128到127效率最高。

如果不去配置虚拟机参数，这个值不会变。配合valueOf(int) 方法，可以节省创建对象造成的资源消耗。
### valueOf(int) 方法
```
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
    	return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

## 该改变缓存值得范围

启动JVM时可以通过
```
-Djava.lang.Integer.IntegerCache.high=xxx
```
就可以改变缓存值的最大值，比如
```
-Djava.lang.Integer.IntegerCache.high=888
```
则会缓存[-888]。 

## 部分方法

### parseInt

```
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}
public static int parseInt(String s, int radix)
            throws NumberFormatException
{
    /*
     * WARNING: This method may be invoked early during VM initialization
     * before IntegerCache is initialized. Care must be taken to not use
     * the valueOf method.
     */

    if (s == null) {
        throw new NumberFormatException("null");
    }

    if (radix < Character.MIN_RADIX) {
        throw new NumberFormatException("radix " + radix +
                                        " less than Character.MIN_RADIX");
    }

    if (radix > Character.MAX_RADIX) {
        throw new NumberFormatException("radix " + radix +
                                        " greater than Character.MAX_RADIX");
    }

    int result = 0;
    boolean negative = false;
    int i = 0, len = s.length();
    int limit = -Integer.MAX_VALUE;
    int multmin;
    int digit;

    if (len > 0) {
        char firstChar = s.charAt(0);
        if (firstChar < '0') { // Possible leading "+" or "-"
            if (firstChar == '-') {
                negative = true;
                limit = Integer.MIN_VALUE;
            } else if (firstChar != '+')
                throw NumberFormatException.forInputString(s);

            if (len == 1) // Cannot have lone "+" or "-"
                throw NumberFormatException.forInputString(s);
            i++;
        }
        multmin = limit / radix;
        while (i < len) {
            // Accumulating negatively avoids surprises near MAX_VALUE
            digit = Character.digit(s.charAt(i++),radix);
            if (digit < 0) {
                throw NumberFormatException.forInputString(s);
            }
            if (result < multmin) {
                throw NumberFormatException.forInputString(s);
            }
            result *= radix;
            if (result < limit + digit) {
                throw NumberFormatException.forInputString(s);
            }
            result -= digit;
        }
    } else {
        throw NumberFormatException.forInputString(s);
    }
    return negative ? result : -result;
}
```

有两个parseInt方法，这里就是java的overload(重载)了，以后再介绍，第一个参数是需要转换的字符串，第二个参数表示进制数（比如2,8,10）。

**源码里已经给出了例子：**

```
* parseInt("0", 10) returns 0
* parseInt("473", 10) returns 473
* parseInt("+42", 10) returns 42
* parseInt("-0", 10) returns 0
* parseInt("-FF", 16) returns -255
* parseInt("1100110", 2) returns 102
* parseInt("2147483647", 10) returns 2147483647
* parseInt("-2147483648", 10) returns -2147483648
* parseInt("2147483648", 10) throws a NumberFormatException
* parseInt("99", 8) throws a NumberFormatException
* parseInt("Kona", 10) throws a NumberFormatException
* parseInt("Kona", 27) returns 411787d
```

但是`Integer.parseInt("10000000000",10)`，会抛出`java.lang.NumberFormatException`异常,因为超过了Integer的最大的范围了。 

该方法的逻辑：

1.判断字符串不为空，并且传进来进制参数在 Character.MIN_RADIX ，Charcater.MAX_RADIX 之间，即2和36。

2.判断输入的字符串长度必须大于0，再根据第一个字符串可能是数字或负号或正号进行处理。

3.之后就是核心逻辑了，字符串转换数字。

例如

127 转换成8进制 1*8*8+2*8+7*1=87

127转换成十进制 1*10*10+2*10+7*1=127

从左到右遍历字符串，然后乘上进制数，再加上下一个字符，接着再乘以进制数，再加上下个字符，不断重复，直到最后一个字符。

这里也是通过计算负数再添加符号位的方式避免Integer.MIN_VALUE单独处理的问题，，因为负数`Integer.MIN_VALUE`变化为正数时会导致数值溢出，所以全部都用负数来运算。这里使用了一个mulmin变量来避免数字溢出的问题，单纯的使用正负号不能准确判断是否溢出(一次乘法导致溢出的结果符号不确定)。 

### 构造函数

```
public Integer(int value) {
        this.value = value;
    }

public Integer(String s) throws NumberFormatException {
        this.value = parseInt(s, 10);
    }
```

这两种构造函数，一个是传int类型，一个是String类型。第二种是通过调用parseInt方法进行转换的。

### getChars方法

```
static void getChars(int i, int index, char[] buf) {
    int q, r;
    int charPos = index;
    char sign = 0;

    if (i < 0) {
        sign = '-';
        i = -i;
    }

    // Generate two digits per iteration
    while (i >= 65536) {
        q = i / 100;
    // really: r = i - (q * 100);
        r = i - ((q << 6) + (q << 5) + (q << 2));
        i = q;
        buf [--charPos] = DigitOnes[r];
        buf [--charPos] = DigitTens[r];
    }

    // Fall thru to fast mode for smaller numbers
    // assert(i <= 65536, i);
    for (;;) {
        q = (i * 52429) >>> (16+3);
        r = i - ((q << 3) + (q << 1));  // r = i-(q*10) ...
        buf [--charPos] = digits [r];
        i = q;
        if (i == 0) break;
    }
    if (sign != 0) {
        buf [--charPos] = sign;
    }
}
```



该方法主要做的事情是将某个int型数值放到char数组里面。

这里面处理用了一些技巧，int高位的两个字节和低位的两个字节分开处理，

while (i >= 65536)部分就是处理高位的两个字节，大于65536的部分使用了除法，一次生成两位十进制字符，每次处理2位数，其实((q << 6) + (q << 5) + (q << 2))其实等于q*100,DigitTens和DigitOnes数组前面已经讲过它的作用了，用来获取十位和个位。

小于等于65536的部分使用了乘法加位移做成除以10的效果，比如(i * 52429) >>> (16+3)其实约等于i/10，((q << 3) + (q << 1))其实等于q*10，然后再通过digits数组获取到对应的字符。可以看到低位处理时它尽量避开了除法，取而代之的是用乘法和右移来实现，可见除法是一个比较耗时的操作，比起乘法和移位。另外也可以看到能用移位和加法来实现乘法的地方也尽量不用乘法，这也说明乘法比起它们更加耗时。而高位处理时没有用移位是因为做乘法后可能会溢出。 

### toString方法

```
public static String toString(int i) {
        if (i == Integer.MIN_VALUE)
            return "-2147483648";
        int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
        char[] buf = new char[size];
        getChars(i, size, buf);
        return new String(buf, true);
    }
public String toString() {
        return toString(value);
    }
public static String toString(int i, int radix) {
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;

        if (radix == 10) {
            return toString(i);
        }

        char buf[] = new char[33];
        boolean negative = (i < 0);
        int charPos = 32;

        if (!negative) {
            i = -i;
        }

        while (i <= -radix) {
            buf[charPos--] = digits[-(i % radix)];
            i = i / radix;
        }
        buf[charPos] = digits[-i];

        if (negative) {
            buf[--charPos] = '-';
        }

        return new String(buf, charPos, (33 - charPos));
    }
```

这里有三个toString 方法。

第一个toString方法很简单，就是先用stringSize得到数字是多少位，再用getChars获取数字对应的char数组，最后返回一个String类型。

第二个更简单了，就是调用第一个toString方法。

第三个toString方法是带了进制信息的，它会转换成对应进制的字符串。不在2到36进制范围之间的都会按照10进制处理。

## valueOf方法

```
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
public static Integer valueOf(String s) throws NumberFormatException {
        return Integer.valueOf(parseInt(s, 10));
    }
public static Integer valueOf(String s, int radix) throws NumberFormatException {
        return Integer.valueOf(parseInt(s,radix));
    }
复制代码
```



第一个valueOf中，因为IntegerCache缓存了【low,high】值的Integer对象，对于在【-128,127】范围内的直接从IntegerCache的数组中获取对应的Integer对象即可，而在范围外的则需要重新实例化了。

第二个和第三个都是调用第一个valueOf方法，只不过调用的parseInt（）的方法不一样。

### decode方法

```
public static Integer decode(String nm) throws NumberFormatException {
    int radix = 10;
    int index = 0;
    boolean negative = false;
    Integer result;

    if (nm.length() == 0)
        throw new NumberFormatException("Zero length string");
    char firstChar = nm.charAt(0);
    // Handle sign, if present
    if (firstChar == '-') {
        negative = true;
        index++;
    } else if (firstChar == '+')
        index++;

    // Handle radix specifier, if present
    if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
        index += 2;
        radix = 16;
    }
    else if (nm.startsWith("#", index)) {
        index ++;
        radix = 16;
    }
    else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
        index ++;
        radix = 8;
    }

    if (nm.startsWith("-", index) || nm.startsWith("+", index))
        throw new NumberFormatException("Sign character in wrong position");

    try {
        result = Integer.valueOf(nm.substring(index), radix);
        result = negative ? Integer.valueOf(-result.intValue()) : result;
    } catch (NumberFormatException e) {
        // If number is Integer.MIN_VALUE, we'll end up here. The next line
        // handles this case, and causes any genuine format error to be
        // rethrown.
        String constant = negative ? ("-" + nm.substring(index))
                                   : nm.substring(index);
        result = Integer.valueOf(constant, radix);
    }
    return result;
}
```

decode方法主要作用是对字符串进行解码。

默认会处理成十进制，0开头的都会处理成8进制，0x和#开头的都会处理成十六进制。

### xxxvalue方法（byteValue，shortValue，intValue，longValue，floatValue，doubleValue）

```
public byte byteValue() {
        return (byte)value;
    }
public short shortValue() {
        return (short)value;
    }
public int intValue() {
        return value;
    }
public long longValue() {
        return (long)value;
    }
public float floatValue() {
        return (float)value;
    }
public double doubleValue() {
        return (double)value;
    }
```



其实就是转换成对应的类型

### hashCode方法

```
public int hashCode() {
    return value;
}
```



就是返回int类型的值

### equals方法

```
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
复制代码
```



比较是否相同之前会将int类型通过valueof转换成Integer类型，equals本质就是值得比较

### compare方法

```
public static int compare(int x, int y) {      

    return (x < y) ? -1 : ((x == y) ? 0 : 1);
}复制代码
```

x小于y则返回-1，相等则返回0，否则返回1。

bitCount方法

```
   public static int bitCount(int i) {
        // HD, Figure 5-2
        i = i - ((i >>> 1) & 0x55555555);
        i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
        i = (i + (i >>> 4)) & 0x0f0f0f0f;
        i = i + (i >>> 8);
        i = i + (i >>> 16);
        return i & 0x3f;
    }
复制代码
```

该方法主要用于计算二进制数中1的个数。

这些都是移位和加减操作。

`0x55555555`等于`01010101010101010101010101010101`，`0x33333333`等于`110011001100110011001100110011`，`0x0f0f0f0f`等于`1111000011110000111100001111`。

先每**两位**一组统计看有多少个1，比如`10011111`则每两位有1、1、2、2个1，记为`01011010`，然后再算每**四位**一组看有多少个1，而`01011010`则每四位有2、4个1，记为`00100100`，      接着每**八位**一组就为`00000110`，接着**16位，32位**，最终在与`0x3f`进行与运算，得到的数即为1的个数。

### highestOneBit方法

```
    public static int highestOneBit(int i) {
        // HD, Figure 3-1
        i |= (i >>  1);
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        return i - (i >>> 1);
    }
// 随便一个例子，不用管最高位之后有多少个1，都会被覆盖
// 00010000 00000000 00000000 00000000      raw
// 00011000 00000000 00000000 00000000      i | (i >> 1)
// 00011110 00000000 00000000 00000000      i | (i >> 2)
// 00011111 11100000 00000000 00000000      i | (i >> 4)
// 00011111 11111111 11100000 00000000      i | (i >> 8)
// 00011111 11111111 11111111 11111111      i | (i >> 16)
// 00010000 0000000 00000000 00000000       i - (i >>>1)
```

这个方法是返回最高位的1，其他都是0的值。将i右移一位再或操作，则最高位1的右边也为1了，接着再右移两位并或操作，则右边1+2=3位都为1了，接着1+2+4=7位都为1，直到1+2+4+8+16=31都为1，最后用`i - (i >>> 1)`自然得到最终结果。

### lowestOneBit方法

```
    public static int lowestOneBit(int i) {
        // HD, Section 2-1
        return i & -i;
    }
// 例子
// 00001000 10000100 10001001 00101000    i
// 11110111 01111011 01110110 11011000    -i
// 00000000 00000000 00000000 00001000  i & -i
```

与highestOneBit方法对应，lowestOneBit获取最低位1，其他全为0的值。这个操作较简单，先取负数，这个过程需要对正数的i取反码然后再加1，得到的结果和i进行与操作，刚好就是最低位1其他为0的值了。

### numberOfLeadingZeros方法

```
    public static int numberOfLeadingZeros(int i) {
        // HD, Figure 5-6
        if (i == 0)
            return 32;
        int n = 1;
        if (i >>> 16 == 0) { n += 16; i <<= 16; }
        if (i >>> 24 == 0) { n +=  8; i <<=  8; }
        if (i >>> 28 == 0) { n +=  4; i <<=  4; }
        if (i >>> 30 == 0) { n +=  2; i <<=  2; }
        n -= i >>> 31;
        return n;
    }
// 方法很巧妙， 类似于二分法。不断将数字左移缩小范围。例子用最差情况：
// i: 00000000 00000000 00000000 00000001         n = 1
// i: 00000000 00000001 00000000 00000000         n = 17
// i: 00000001 00000000 00000000 00000000         n = 25
// i: 00010000 00000000 00000000 00000000         n = 29
// i: 01000000 00000000 00000000 00000000         n = 31
// i >>>31 == 0
// return 31
```

该方法返回i的二进制从头开始有多少个0。i为0的话则有32个0。这里处理其实是体现了二分查找思想的，先看高16位是否为0，是的话则至少有16个0，否则左移16位继续往下判断，接着右移24位看是不是为0，是的话则至少有16+8=24个0，直到最后得到结果。

### numberOfTrailingZeros方法

```
    public static int numberOfTrailingZeros(int i) {
        // HD, Figure 5-14
        int y;
        if (i == 0) return 32;
        int n = 31;
        y = i <<16; if (y != 0) { n = n -16; i = y; }
        y = i << 8; if (y != 0) { n = n - 8; i = y; }
        y = i << 4; if (y != 0) { n = n - 4; i = y; }
        y = i << 2; if (y != 0) { n = n - 2; i = y; }
        return n - ((i << 1) >>> 31);
    }
// 与求开头多少个0类似，也是用了二分法，先锁定1/2, 再锁定1/4，1/8，1/16，1/32。
// i: 11111111 11111111 11111111 11111111    n: 31
// i: 11111111 11111111 00000000 00000000    n: 15
// i: 11111111 00000000 00000000 00000000    n: 7
// i: 11110000 00000000 00000000 00000000    n: 3
// i: 11000000 00000000 00000000 00000000    n: 1
// i: 10000000 00000000 00000000 00000000    n: 0
```

与前面的numberOfLeadingZeros方法对应，该方法返回i的二进制从尾开始有多少个0。它的思想和前面的类似，也是基于二分查找思想，详细步骤不再赘述。

### reerse方法

```
public static int reverse(int i) {
        i = (i & 0x55555555) << 1 | (i >>> 1) & 0x55555555;
        i = (i & 0x33333333) << 2 | (i >>> 2) & 0x33333333;
        i = (i & 0x0f0f0f0f) << 4 | (i >>> 4) & 0x0f0f0f0f;
        i = (i << 24) | ((i & 0xff00) << 8) |
            ((i >>> 8) & 0xff00) | (i >>> 24);
        return i;
    }
```

该方法即是将i进行反转，反转就是第1位与第32位对调，第二位与第31位对调，以此类推。它的核心思想是先将相邻两位进行对换，比如10100111对换01011011，接着再将相邻四位进行对换，对换后为10101101，接着将相邻八位进行对换，最后把32位中中间的16位对换，然后最高8位再和最低8位对换。

### toHexString和toOctalString方法

```
public static String toHexString(int i) {
        return toUnsignedString0(i, 4);
    }
public static String toOctalString(int i) {
        return toUnsignedString0(i, 3);
    }
private static String toUnsignedString0(int val, int shift) {
        int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
        int chars = Math.max(((mag + (shift - 1)) / shift), 1);
        char[] buf = new char[chars];

        formatUnsignedInt(val, shift, buf, 0, chars);

        return new String(buf, true);
    }
static int formatUnsignedInt(int val, int shift, char[] buf, int offset, int len) {
        int charPos = len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            buf[offset + --charPos] = Integer.digits[val & mask];
            val >>>= shift;
        } while (val != 0 && charPos > 0);

        return charPos;
    }
```

这两个方法类似，合到一起讲。看名字就知道转成8进制和16进制的字符串。可以看到都是间接调用toUnsignedString0方法，该方法会先计算转换成对应进制需要的字符数，然后再通过formatUnsignedInt方法来填充字符数组，该方法做的事情就是使用进制之间的转换方法（前面有提到过）来获取对应的字符。

# 总结
理解到int和Integer的却别，IntegerCache内部类的作用，还有就是`==`和`equal`在上面情况下使用才正确，怎么灵活适应Integer里面的方法，进制之间的转换等等。