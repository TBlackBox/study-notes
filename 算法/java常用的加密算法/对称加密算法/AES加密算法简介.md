# 简介
AES是一个对称密码，旨在取代DES成为广泛使用的标准。

AES加密过程涉及到 4 种操作，分别是`字节替代`、`行移位`、`列混淆`和`轮密钥加`。解密过程分别为对应的逆操作。由于每一步操作都是可逆的，按照相反的顺序进行解密即可恢复明文。加解密中每轮的密钥分别由初始密钥扩展得到。算法中 16 个字节的明文、密文和轮密钥都以一个 4x4 的矩阵表示。 

AES是一种十分成熟、安全、国际通用的对称密码的加密解密算法，供AES加密解密的重要参数就是密钥。这个密钥只是一个隨机字符串，通常是`128`位或`256`位字长。

# AES五种加密模式（CBC、ECB、CTR、OCF、CFB）
一般的加密通常都是块加密，如果要加密超过块大小的数据，就需要涉及填充和链加密模式。

分组密码有五种工作体制：
1. 电码本模式（Electronic Codebook Book (ECB)）；
2. 密码分组链接模式（Cipher Block Chaining (CBC)）；
3. 计算器模式（Counter (CTR)）；
4. 密码反馈模式（Cipher FeedBack (CFB)）；
5. 输出反馈模式（Output FeedBack (OFB)）。


## 电码本模式（Electronic Codebook Book (ECB)）
这种模式是将整个明文分成若干段相同的小段，然后对每一小段进行加密。
- 优点:
  - 简单；
  - 有利于并行计算；
  - 误差不会被传送；
- 缺点:
  - 不能隐藏明文的模式；
  - 可能对明文进行主动攻击；

## 密码分组链接模式 Cipher Block Chaining (CBC)
这种模式是先将明文切分成若干小段，然后每一小段与初始块或者上一段的密文段进行异或运算后，再与密钥进行加密。
- 优点：
  - 不容易主动攻击,安全性好于ECB,适合传输长度长的报文,是SSL、IPSec的标准。
- 缺点：
  - 不利于并行计算；
  - 误差传递；
  - 需要初始化向量IV

  ## 计算器模式Counter (CTR)
  计算器模式不常见，在CTR模式中， 有一个自增的算子，这个算子用密钥加密之后的输出和明文异或的结果得到密文，相当于一次一密。这种加密方式简单快速，安全可靠，而且可以并行加密，但是 在计算器不能维持很长的情况下，密钥只能使用一次 。

- 优点：
  - 无填
  - 同明文不同密
  - 每个块单独运算，适合并行运算。
- 缺点：
  - 可能导致明文攻击。

# 密码反馈模式（Cipher FeedBack (CFB)
- 优点：
  - 隐藏了明文模式;
  - 分组密码转化为流模式;
  - 可以及时加密传送小于分组的数据;
- 缺点:
  - 不利于并行计算;
  - 误差传送：一个明文单元损坏影响多个单元;
  - 唯一的IV;。

## 输出反馈模式Output FeedBack (OFB)
- 优点：
  - 同明文不同密文，分组密钥转换为流密码。
- 缺点：
  - 串行运算不利并行
  - 传输错误可能导致后续传输块错误。


  # JAVA工具类

```java
package algorithm;

import java.math.BigInteger;
import java.util.Base64;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.spec.SecretKeySpec;

/**
 * AES加解密算法
 */
public final class AESUtil {
    //默认密钥 
    private static final String KEY = "febcb12e13751f21";
    //算法
    private static final String ALGORITHMSTR = "AES/ECB/PKCS5Padding";

    /**
     * aes解密
     *
     * @param encrypt 内容
     */
    public static String decrypt(String encrypt) {
        return decrypt(encrypt, KEY);
    }

    /**
     * aes加密
     *
     * @param content
     */
    public static String encrypt(String content) {
        return encrypt(content, KEY);
    }

    /**
     * 将byte[]转为各种进制的字符串
     *
     * @param bytes byte[]
     * @param radix 可以转换进制的范围，从Character.MIN_RADIX到Character.MAX_RADIX，超出范围后变为10进制
     * @return 转换后的字符串
     */
    public static String binary(byte[] bytes, int radix) {
        return new BigInteger(1, bytes).toString(radix);// 这里的1代表正数  
    }

    /**
     * base 64 encode
     *
     * @param bytes 待编码的byte[]
     * @return 编码后的base 64 code
     */
    public static String base64Encode(byte[] bytes) {
        return Base64.getEncoder().encodeToString(bytes);
    }

    /**
     * base 64 decode
     *
     * @param base64Code 待解码的base 64 code
     * @return 解码后的byte[]
     * @throws Exception
     */
    public static byte[] base64Decode(String base64Code) throws Exception {
        return Base64.getDecoder().decode(base64Code);
    }


    /**
     * AES加密
     *
     * @param content    待加密的内容
     * @param encryptKey 加密密钥
     * @return 加密后的byte[]
     * @throws Exception
     */
    public static byte[] encryptToBytes(String content, String encryptKey) throws Exception {
        KeyGenerator kgen = KeyGenerator.getInstance("AES");
        kgen.init(128);
        Cipher cipher = Cipher.getInstance(ALGORITHMSTR);
        cipher.init(Cipher.ENCRYPT_MODE, new SecretKeySpec(encryptKey.getBytes(), "AES"));
        return cipher.doFinal(content.getBytes("utf-8"));
    }


    /**
     * AES加密为base 64 code
     *
     * @param content    待加密的内容
     * @param encryptKey 加密密钥
     * @return 加密后的base 64 code
     */
    public static String encrypt(String content, String encryptKey) {
        try {
            return base64Encode(encryptToBytes(content, encryptKey));
        } catch (Exception e) {
            e.printStackTrace();
        }
		return null;
    }

    /**
     * AES解密
     *
     * @param encryptBytes 待解密的byte[]
     * @param decryptKey   解密密钥
     * @return 解密后的String
     * @throws Exception
     */
    public static String decryptByBytes(byte[] encryptBytes, String decryptKey) throws Exception {
        KeyGenerator kgen = KeyGenerator.getInstance("AES");
        kgen.init(128);

        Cipher cipher = Cipher.getInstance(ALGORITHMSTR);
        cipher.init(Cipher.DECRYPT_MODE, new SecretKeySpec(decryptKey.getBytes(), "AES"));
        byte[] decryptBytes = cipher.doFinal(encryptBytes);
        return new String(decryptBytes);
    }


    /**
     * 将base 64 code AES解密
     *
     * @param encryptStr 待解密的base 64 code
     * @param decryptKey 解密密钥
     * @return 解密后的string
     */
    public static String decrypt(String encryptStr, String decryptKey) {
        try {
            return decryptByBytes(base64Decode(encryptStr), decryptKey);
        } catch (Exception e) {
        	e.printStackTrace();
        }
		return null;
    }

    public static void main(String[] args) {
        String orgStr = "abcd1234";
        String enStr = encrypt(orgStr);
        System.out.println("enStr: " + enStr); //enStr: SgZUJkAfAkm6cDjOtzw+Gg==
        String deStr = decrypt(enStr);
        System.out.println("deStr: " + deStr); //deStr: abcd1234
    }
}


  ```