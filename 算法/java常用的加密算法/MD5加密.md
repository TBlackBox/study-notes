# 简介
MD5讯息摘要演算法（英语：MD5 Message-Digest Algorithm），一种被广泛使用的密码杂凑函数，可以产生出一个128位元（16位元组）的散列值（hash value），用于确保信息传输完整一致。

# MD5功能
1. 输入任意长度的信息，经过处理，输出为128位的信息（数字指纹）；
2. 不同的输入得到的不同的结果（唯一性）；

# MD5属不属于加密算法

- 认为不属于的人是因为他们觉得不能从密文（散列值）反过来得到原文，即没有解密算法，所以这部分人认为MD5只能属于算法，不能称为加密算法；
- 认为属于的人是因为他们觉得经过MD5处理后看不到原文，即已经将原文加密，所以认为MD5属于加密算法；我个人支持前者，正如认为BASE64算法只能算编码一样。

# MD5算法不可逆

MD5不可逆的原因是其是一种散列函数，使用的是hash算法，在计算过程中原文的部分信息是丢失了的。

不过有个地方值得指出的是，一个MD5理论上的确是可能对应无数多个原文的，因为MD5是有限多个的而原文可以是无数多个。比如主流使用的MD5将任意长度的“字节串映射为一个128bit的大整数。也就是一共有2^128种可能，大概是3.4*10^38，这个数字是有限多个的，而但是世界上可以被用来加密的原文则会有无数的可能性。

不过需要注意的一点是，尽量这是一个理论上的有限对无限，不过问题是这个无限在现实生活中并不完全成立，因为一方面现实中原文的长度往往是有限的（以常用的密码为例，一般人都在20位以内），另一方面目前想要发现两段原文对应同一个MD5（专业的说这叫杂凑冲撞）值非常困难，因此某种意义上来说，在一定范围内想构建MD5值与原文的一一对应关系是完全有可能的。所以对于MD5目前最有效的攻击方式就是彩虹表，具体详情你可以通过谷歌了解。

MD5相当于超损压缩。

# **MD5用途**

1. 防止被篡改：
- 比如发送一个电子文档，发送前，我先得到MD5的输出结果a。然后在对方收到电子文档后，对方也得到一个MD5的输出结果b。如果a与b一样就代表中途未被篡改。
- 比如我提供文件下载，为了防止不法分子在安装程序中添加木马，我可以在网站上公布由安装文件得到的MD5输出结果。
- SVN在检测文件是否在CheckOut后被修改过，也是用到了MD5.

2. 防止直接看到明文：
现在很多网站在数据库存储用户的密码的时候都是存储用户密码的MD5值。这样就算不法分子得到数据库的用户密码的MD5值，也无法知道用户的密码。（比如在UNIX系统中用户的密码就是以MD5（或其它类似的算法）经加密后存储在文件系统中。当用户登录的时候，系统把用户输入的密码计算成MD5值，然后再去和保存在文件系统中的MD5值进行比较，进而确定输入的密码是否正确。通过这样的步骤，系统在并不知道用户密码的明码的情况下就可以确定用户登录系统的合法性。这不但可以避免用户的密码被具有系统管理员权限的用户知道，而且还在一定程度上增加了密码被破解的难度。）

3. 防止抵赖（数字签名）：
这需要一个第三方认证机构。例如A写了一个文件，认证机构对此文件用MD5算法产生摘要信息并做好记录。若以后A说这文件不是他写的，权威机构只需对此文件重新产生摘要信息，然后跟记录在册的摘要信息进行比对，相同的话，就证明是A写的了。这就是所谓的“数字签名”。

# **MD5安全性**

普遍认为MD5是很安全，因为暴力破解的时间是一般人无法接受的。实际上如果把用户的密码MD5处理后再存储到数据库，其实是很不安全的。因为用户的密码是比较短的，而且很多用户的密码都使用生日，手机号码，身份证号码，电话号码等等。或者使用常用的一些吉利的数字，或者某个英文单词。如果我把常用的密码先MD5处理，把数据存储起来，然后再跟你的MD5结果匹配，这时我就有可能得到明文。比如某个MD5破解网站http://www.cmd5.com/default.aspx，所以现在大多数网站密码的策略是强制要求用户使用数字大小写字母的组合的方式提高用户密码的安全度。

# **MD5算法过程**

对MD5算法简要的叙述可以为：MD5以512位分组来处理输入的信息，且每一分组又被划分为16个32位子分组，经过了一系列的处理后，算法的输出由四个32位分组组成，将这四个32位分组级联后将生成一个128位散列值。

第一步、填充：如果输入信息的长度(bit)对512求余的结果不等于448，就需要填充使得对512求余的结果等于448。填充的方法是填充一个1和n个0。填充完后，信息的长度就为N*512+448(bit)；

第二步、记录信息长度：用64位来存储填充前信息长度。这64位加在第一步结果的后面，这样信息长度就变为N*512+448+64=(N+1)*512位。

第三步、装入标准的幻数（四个整数）：标准的幻数（物理顺序）是（A=(01234567)16，B=(89ABCDEF)16，C=(FEDCBA98)16，D=(76543210)16）。如果在程序中定义应该是:
（A=0X67452301L，B=0XEFCDAB89L，C=0X98BADCFEL，D=0X10325476L）。有点晕哈，其实想一想就明白了。

第四步、四轮循环运算：循环的次数是分组的个数（N+1）

1）将每一512字节细分成16个小组，每个小组64位（8个字节）

2）先认识四个线性函数(&是与,|是或,~是非,^是异或)

```
F(X,Y,Z)=(X&Y)|((~X)&Z)
G(X,Y,Z)=(X&Z)|(Y&(~Z))
H(X,Y,Z)=X^Y^Z
I(X,Y,Z)=Y^(X|(~Z))1234
```

3）设Mj表示消息的第j个子分组（从0到15），<<

```
FF(a,b,c,d,Mj,s,ti)表示a=b+((a+F(b,c,d)+Mj+ti)<<<s)
GG(a,b,c,d,Mj,s,ti)表示a=b+((a+G(b,c,d)+Mj+ti)<<<s)
HH(a,b,c,d,Mj,s,ti)表示a=b+((a+H(b,c,d)+Mj+ti)<<<s)
II(a,b,c,d,Mj,s,ti)表示a=b+((a+I(b,c,d)+Mj+ti)<<<s)1234
```

4）四轮运算

```
 第一轮
a=FF(a,b,c,d,M0,7,0xd76aa478)
b=FF(d,a,b,c,M1,12,0xe8c7b756)
c=FF(c,d,a,b,M2,17,0x242070db)
d=FF(b,c,d,a,M3,22,0xc1bdceee)
a=FF(a,b,c,d,M4,7,0xf57c0faf)
b=FF(d,a,b,c,M5,12,0x4787c62a)
c=FF(c,d,a,b,M6,17,0xa8304613)
d=FF(b,c,d,a,M7,22,0xfd469501)
a=FF(a,b,c,d,M8,7,0x698098d8)
b=FF(d,a,b,c,M9,12,0x8b44f7af)
c=FF(c,d,a,b,M10,17,0xffff5bb1)
d=FF(b,c,d,a,M11,22,0x895cd7be)
a=FF(a,b,c,d,M12,7,0x6b901122)
b=FF(d,a,b,c,M13,12,0xfd987193)
c=FF(c,d,a,b,M14,17,0xa679438e)
d=FF(b,c,d,a,M15,22,0x49b40821)

第二轮
a=GG(a,b,c,d,M1,5,0xf61e2562)
b=GG(d,a,b,c,M6,9,0xc040b340)
c=GG(c,d,a,b,M11,14,0x265e5a51)
d=GG(b,c,d,a,M0,20,0xe9b6c7aa)
a=GG(a,b,c,d,M5,5,0xd62f105d)
b=GG(d,a,b,c,M10,9,0x02441453)
c=GG(c,d,a,b,M15,14,0xd8a1e681)
d=GG(b,c,d,a,M4,20,0xe7d3fbc8)
a=GG(a,b,c,d,M9,5,0x21e1cde6)
b=GG(d,a,b,c,M14,9,0xc33707d6)
c=GG(c,d,a,b,M3,14,0xf4d50d87)
d=GG(b,c,d,a,M8,20,0x455a14ed)
a=GG(a,b,c,d,M13,5,0xa9e3e905)
b=GG(d,a,b,c,M2,9,0xfcefa3f8)
c=GG(c,d,a,b,M7,14,0x676f02d9)
d=GG(b,c,d,a,M12,20,0x8d2a4c8a)

第三轮
a=HH(a,b,c,d,M5,4,0xfffa3942)
b=HH(d,a,b,c,M8,11,0x8771f681)
c=HH(c,d,a,b,M11,16,0x6d9d6122)
d=HH(b,c,d,a,M14,23,0xfde5380c)
a=HH(a,b,c,d,M1,4,0xa4beea44)
b=HH(d,a,b,c,M4,11,0x4bdecfa9)
c=HH(c,d,a,b,M7,16,0xf6bb4b60)
d=HH(b,c,d,a,M10,23,0xbebfbc70)
a=HH(a,b,c,d,M13,4,0x289b7ec6)
b=HH(d,a,b,c,M0,11,0xeaa127fa)
c=HH(c,d,a,b,M3,16,0xd4ef3085)
d=HH(b,c,d,a,M6,23,0x04881d05)
a=HH(a,b,c,d,M9,4,0xd9d4d039)
b=HH(d,a,b,c,M12,11,0xe6db99e5)
c=HH(c,d,a,b,M15,16,0x1fa27cf8)
d=HH(b,c,d,a,M2,23,0xc4ac5665)

第四轮
a=II(a,b,c,d,M0,6,0xf4292244)
b=II(d,a,b,c,M7,10,0x432aff97)
c=II(c,d,a,b,M14,15,0xab9423a7)
d=II(b,c,d,a,M5,21,0xfc93a039)
a=II(a,b,c,d,M12,6,0x655b59c3)
b=II(d,a,b,c,M3,10,0x8f0ccc92)
c=II(c,d,a,b,M10,15,0xffeff47d)
d=II(b,c,d,a,M1,21,0x85845dd1)
a=II(a,b,c,d,M8,6,0x6fa87e4f)
b=II(d,a,b,c,M15,10,0xfe2ce6e0)
c=II(c,d,a,b,M6,15,0xa3014314)
d=II(b,c,d,a,M13,21,0x4e0811a1)
a=II(a,b,c,d,M4,6,0xf7537e82)
b=II(d,a,b,c,M11,10,0xbd3af235)
c=II(c,d,a,b,M2,15,0x2ad7d2bb)
d=II(b,c,d,a,M9,21,0xeb86d391)1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071
```

5）每轮循环后，将A，B，C，D分别加上a，b，c，d，然后进入下一循环。

# Java工具类
```java
public class Md5 {


	/**
	 * MD5加密后的字节以十六进制形式字符串返回
	 * @param cleartext
	 * @return
	 */
	public static String toMd5(String cleartext) {  
		return toMd5(cleartext.getBytes());  
	}  

	public static String toMd5(ByteArrayOutputStream baos){
		return toMd5(baos.toByteArray());  
	}

	public static String toMd5(byte[] data){
		try {
			// Create MD5 Hash  
			MessageDigest digest = MessageDigest.getInstance("MD5");  
			digest.update(data);  
			byte messageDigest[] = digest.digest();  
			return HexUtil.toHex(messageDigest);
		} catch (NoSuchAlgorithmException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}  
		return "";  
	}
}
```