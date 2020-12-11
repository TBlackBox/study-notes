# 简介
## DES算法的历史　　
美国国家标准局1973年开始研究除国防部外的其它部门的计算机系统的数据加密标准，于1973年5月15日和1974年8月27日先后两次向公众发出了征求加密算法的公告。  
加密算法要达到的目的（通常称为DES 密码算法要求）主要为以下四点：
- 提供高质量的数据保护，防止数据未经授权的泄露和未被察觉的修改； 
- 具有相当高的复杂性，使得破译的开销超过可能获得的利益，同时又要便于理解和掌握； 
- DES密码体制的安全性应该不依赖于算法的保密，其安全性仅以加密密钥的保密为基础； 
- 实现经济，运行有效，并且适用于多种完全不同的应用。        

1977年1月，美国政府颁布：采纳IBM公司设计的方案作为非机密数据的正式数据加密标准（DES   Data Encryption Standard）。

# DES 的运用
目前在国内，随着三金工程尤其是金卡工程的启动，DES算法在POS、ATM、磁卡及智能卡（IC卡）、加油站、高速公路收费站等领域被广泛应用，以此来实现关键数据的保密，如信用卡持卡人的PIN的加密传输，IC卡与POS间的双向认证、金融交易数据包的MAC校验等，均用到DES算法。


# DES的原理简介

DES算法的入口参数有三个：`Key`、`Data`、`Mode`。

- Key为8个字节共64位，是DES算法的工作密钥；

- Data也为8个字节64位，是要被加密或被解密的数据；

- Mode为DES的工作方式，有两种：加密或解密。

DES是一个基于组块的加密算法，这意味着无论输入还是输出都是64位长度的。也就是说DES产生了一种最多264种的变换方法。每个64位的区块被分为2个32位的部分，左半部分L和右半部分R。（这种分割只在特定的操作中进行。）


DES（数据加密标准）算法主要采用替换和移位的方式进行加密，它用56位（64位密钥只有56位有效）对64位二进制数据块进行加密，每次加密对64位的输入数据进行16轮编码，经过一系列替换和移位后，输入的64位原数据转换成完全不同的64位输出数据。

DES算法是这样工作的：如Mode为加密，则用Key 去把数据Data进行加密， 生成Data的密码形式（64位）作为DES的输出结果；如Mode为解密，则用Key去把密码形式的数据Data解密，还原为Data的明码形式（64位）作为DES的输出结果。在通信网络的两端，双方约定一致的Key，在通信的源点用Key对核心数据进行DES加密，然后以密码形式在公共通信网（如电话网）中传输到通信网络的终点，数据到达目的地后，用同样的Key对密码数据进行解密，便再现了明码形式的核心数据。这样，便保证了核心数据（如PIN、MAC等）在公共通信网中传输的安全性和可靠性。

# DES的破解
PostScript：
DES 被证明是可以破解的，明文+密钥=密文，这个公式只要知道任何两个，就可以推导出第三个

在已经知道明文和对应密文的情况下，通过穷举和暴力破解是可以破解DES的。

# Java 工具类
该工具类使用了base64进行编码和解码
```java
package algorithm;

import java.io.UnsupportedEncodingException;
import java.security.SecureRandom;
import java.util.Base64;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;

public class DesUtil {
	
	private final static String ENCODE = "GBK";

  
	public static String encrypt(String contant,String password) {
		try {
			return Base64.getEncoder().encodeToString(encrypt(contant.getBytes(ENCODE),password));
		} catch (UnsupportedEncodingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
	}
	
	/**
	 * 加密
	 * @param datasource
	 * @param password
	 * @return
	 */
    public static byte[] encrypt(byte[] datasource, String password) {
        try {
            SecureRandom random = new SecureRandom();
            DESKeySpec desKey = new DESKeySpec(password.getBytes(ENCODE));
            //创建一个密匙工厂，然后用它把DESKeySpec转换成
            SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
            SecretKey securekey = keyFactory.generateSecret(desKey);
            //Cipher对象实际完成加密操作
            Cipher cipher = Cipher.getInstance("DES");
            //用密匙初始化Cipher对象,ENCRYPT_MODE用于将 Cipher 初始化为加密模式的常量
            cipher.init(Cipher.ENCRYPT_MODE, securekey, random);
            //现在，获取数据并加密 正式执行加密操作
            //按单部分操作加密或解密数据，或者结束一个多部分操作
            return cipher.doFinal(datasource);
        } catch (Throwable e) {
        	System.out.println("加密错误，错误信息：" + e.toString());
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 解密
     * @param src
     * @param password
     * @return
     * @throws Exception
     */
    public static byte[] decrypt(byte[] src, String password) throws Exception {
        // DES算法要求有一个可信任的随机数源
        SecureRandom random = new SecureRandom();
        // 创建一个DESKeySpec对象
        DESKeySpec desKey = new DESKeySpec(password.getBytes(ENCODE));
        // 创建一个密匙工厂
        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");//返回实现指定转换的 Cipher 对象
        // 将DESKeySpec对象转换成SecretKey对象
        SecretKey securekey = keyFactory.generateSecret(desKey);
        // Cipher对象实际完成解密操作
        Cipher cipher = Cipher.getInstance("DES");
        // 用密匙初始化Cipher对象
        cipher.init(Cipher.DECRYPT_MODE, securekey, random);
        // 真正开始解密操作
        return cipher.doFinal(src);
    }
    
    public static String decrypt(String contant,String password) {
    	try {
			return new String(decrypt(Base64.getDecoder().decode(contant.getBytes()), password));
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
    }
    
    
    public static void main(String[] args) {
		
    	String contant = "I LOVE YOU";
    	//密码保存8位
    	String password = "12345678";
    	
    	String encryptContant = encrypt(contant, password);
    	System.out.println(encryptContant); //6ANJS1RDMkhlDvhVr3Z5NQ==
    	
    	String decryptContant = decrypt(encryptContant, password);
    	System.out.println(decryptContant); //I LOVE YOU
    	
	}

}
```