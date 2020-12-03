# 简介
Guava是一种基于开源的Java库，其中包含谷歌正在由他们很多项目使用的很多核心库。这个库是为了方便编码，并减少编码错误。这个库提供用于集合，缓存，支持原语，并发性，常见注解，字符串处理，I/O和验证的实用方法。

Guava 的好处：

- 准化 - Guava库是由谷歌托管。

- 高效 - 可靠，快速和有效的扩展JAVA标准库

- 优化 -Guava库经过高度的优化。

- 函数式编程 -增加JAVA功能和处理能力。

- 实用程序 - 提供了经常需要在应用程序开发的许多实用程序类。

- 验证 -提供标准的故障安全验证机制。

- 最佳实践 - 强调最佳的做法。

当然，guava 对布隆过滤器也有实现，


# BloomFilter类

## 相关的属性
```java
/** The bit set of the BloomFilter (not necessarily power of 2!) */
private final LockFreeBitArray bits;

/** Number of hashes per element */
private final int numHashFunctions;

/** The funnel to translate Ts to bytes */
private final Funnel<? super T> funnel;

/** The strategy we employ to map an element T to {@code numHashFunctions} bit indexes. */
private final Strategy strategy;

```
- bits长度为m的位数组，采用LockFreeBitArray类型做了封装。
- numHashFunctions即哈希函数的个数k。
- funnel是Funnel接口实现类的实例，它用于将任意类型T的输入数据转化为Java基本类型的数据（byte、int、char等等）。这里是会转化为byte。
- strategy是布隆过滤器的哈希策略，即数据如何映射到位数组，其具体方法在BloomFilterStrategies枚举中。

## 构造函数
这个类的构造方法是私有的。要创建它的实例，应该通过公有的create()方法。它一共有5种重载方法，具体的构造函数可以看源码。

最终调用的构造函数
```java
 private BloomFilter(
      LockFreeBitArray bits, int numHashFunctions, Funnel<? super T> funnel, Strategy strategy) {
    checkArgument(numHashFunctions > 0, "numHashFunctions (%s) must be > 0", numHashFunctions);
    checkArgument(
        numHashFunctions <= 255, "numHashFunctions (%s) must be <= 255", numHashFunctions);
    this.bits = checkNotNull(bits);
    this.numHashFunctions = numHashFunctions;
    this.funnel = checkNotNull(funnel);
    this.strategy = checkNotNull(strategy);
  }
```
### 构造函数参数解释
- funnel是插入数据的Funnel。
- expectedInsertions是期望插入的元素总个数n。
- fpp即期望假阳性率p。
- strategy即哈希策略。

由上可知，位数组的长度m和哈希函数的个数k分别通过optimalNumOfBits()方法和optimalNumOfHashFunctions()方法来估计。


# 哈希策略
在BloomFilterStrategies枚举中定义了两种哈希策略，都基于著名的MurmurHash算法，分别是MURMUR128_MITZ_32和MURMUR128_MITZ_64。前者是一个简化版。

## 方法
- put() 方法负责向布隆过滤器中插入元素。
- mightContain()方法负责判断元素是否存在。
- writeTo(OutputStream out) 写入一个文件，放入OSS、S3这类对象存储中。
- readFrom(InputStream in, Funnel<? super T> funnel)从文件中读。
### MURMUR128_MITZ_64流程
1. 使用MurmurHash算法对funnel的输入数据进行散列，得到128bit（16B）的字节数组。
2. 取低8字节作为第一个哈希值hash1，取高8字节作为第二个哈希值hash2。
3. 进行k次循环，每次循环都用hash1与hash2的复合哈希做散列，然后对m取模，将位数组中的对应比特设为1。


# 案列
1. 引入guava包
```java
<dependency>
   <groupId>com.google.guava</groupId>
   <artifactId>guava</artifactId>
   <version>30.0-jre</version>
</dependency>
```

2. 案列代码
```java
import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels;

public class GuavaBloomFilter {

	public static void main(String[] args) {
			//预计包含的数据量
			int expectedInsertions = 100000000;
			
			//允许的误差值
			double fpp = 0.01;
		
	        BloomFilter<Integer> bloomFilter = BloomFilter.create(Funnels.integerFunnel(), expectedInsertions, fpp);
	        for (int i = 0; i < 100000000; i++) {
	            bloomFilter.put(i);
	        }
	        System.out.println(bloomFilter.mightContain(100)); //true
	        System.out.println(bloomFilter.mightContain(100000001)); //false
	    }
}

```

# 总结
guava里面实现的布隆过滤器适合单机环境，在分布式环境中，分布式环境中，布隆过滤器肯定还需要考虑是可以共享的资源，这时候我们会想到 Redis，Redis 也实现了布隆过滤器。