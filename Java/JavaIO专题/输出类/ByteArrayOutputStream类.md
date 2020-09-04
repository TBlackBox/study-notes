# 简介
ByteArrayOutputStream 是字节数组输出流。它继承于OutputStream。

ByteArrayOutputStream 中的数据被写入一个 byte 数组。缓冲区会随着数据的不断写入而自动增长。可使用 toByteArray() 和 toString() 获取数据。

## 函数列表
```
// 构造函数
ByteArrayOutputStream()
ByteArrayOutputStream(int size)

             void    close()
synchronized void    reset()
             int     size()
synchronized byte[]  toByteArray()
             String  toString(int hibyte)
             String  toString(String charsetName)
             String  toString()
synchronized void    write(byte[] buffer, int offset, int len)
synchronized void    write(int oneByte)
synchronized void    writeTo(OutputStream out)
```

*** 说明:***
ByteArrayOutputStream实际上是将字节数据写入到“字节数组”中去。

1.  通过ByteArrayOutputStream()创建的“字节数组输出流”对应的字节数组大小是32。
2.  通过ByteArrayOutputStream(int size) 创建“字节数组输出流”，它对应的字节数组大小是size。
3.  write(int oneByte)的作用将int类型的oneByte换成byte类型，然后写入到输出流中。
(04) write(byte[] buffer, int offset, int len) 是将字节数组buffer写入到输出流中，offset是从buffer中读取数据的起始偏移位置，len是读取的长度。
4. writeTo(OutputStream out) 将该“字节数组输出流”的数据全部写入到“输出流out”中。

## 测试
```
public class ByteArrayOutputStreamTest {

    private static final int LEN = 5;
    // 对应英文字母“abcddefghijklmnopqrsttuvwxyz”
    private static final byte[] ArrayLetters = {
        0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B, 0x6C, 0x6D, 0x6E, 0x6F,
        0x70, 0x71, 0x72, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78, 0x79, 0x7A
    };

    public static void main(String[] args) {
        //String tmp = new String(ArrayLetters);
        //System.out.println("ArrayLetters="+tmp);

        tesByteArrayOutputStream() ;
    }

    /**
     * ByteArrayOutputStream的API测试函数
     */
    private static void tesByteArrayOutputStream() {
        // 创建ByteArrayOutputStream字节流
        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        // 依次写入“A”、“B”、“C”三个字母。0x41对应A，0x42对应B，0x43对应C。
        baos.write(0x41);
        baos.write(0x42);
        baos.write(0x43);
        System.out.printf("baos=%s\n", baos);

        // 将ArrayLetters数组中从“3”开始的后5个字节写入到baos中。
        // 即对应写入“0x64, 0x65, 0x66, 0x67, 0x68”，即“defgh”
        baos.write(ArrayLetters, 3, 5);
        System.out.printf("baos=%s\n", baos);

        // 计算长度
        int size = baos.size();
        System.out.printf("size=%s\n", size);

        // 转换成byte[]数组
        byte[] buf = baos.toByteArray();
        String str = new String(buf);
        System.out.printf("str=%s\n", str);

        // 将baos写入到另一个输出流中
        try {
            ByteArrayOutputStream baos2 = new ByteArrayOutputStream();
            baos.writeTo((OutputStream)baos2);
            System.out.printf("baos2=%s\n", baos2);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```