# 简介
ByteArrayInputStream 包含一个内部缓冲区，该缓冲区包含从流中读取的字节。内部计数器跟踪 read 方法要提供的下一个字节。ByteArrayOutputStream实现了一个输出流，其中的数据被写入一个 byte 数组。缓冲区会随着数据的不断写入而自动增长。可使用 toByteArray()和 toString()获取数据。

# 提供的函数
```
// 构造函数
ByteArrayInputStream(byte[] buf)
ByteArrayInputStream(byte[] buf, int offset, int length)

synchronized int         available() //能否读取字节流的下一字节
void                     close() //关闭字节流
synchronized void        mark(int readlimit) //保存当前位置
boolean                  markSupported() //是否支持mark
synchronized int         read() //读取下一字节
synchronized int         read(byte[] buffer, int offset, int length) //将字节流写入buffer数组
synchronized void        reset() //重置索引到mark位置
synchronized long        skip(long byteCount) //跳过n个字节
}
```

## 案列
```
public class TestByteArray {
    // 对应英文字母“abcddefghijklmnopqrsttuvwxyz”
    private final byte[] ArrayLetters = {
            0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B, 0x6C, 0x6D, 0x6E, 0x6F,
            0x70, 0x71, 0x72, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78, 0x79, 0x7A
    };

    public void testByteArrayInputStream() {
        //创建字节流,以ArrayLetters初始化
        ByteArrayInputStream inputStream = new ByteArrayInputStream(ArrayLetters);

        //读取5个字节
        int i = 0;
        System.out.print("前5个字节为: ");
        while (i++ < 5) {
            //是否可读
            if (inputStream.available() >= 0) {
                int buf = inputStream.read();
                System.out.printf("0x%s ", Integer.toHexString(buf));
            }
        }
        System.out.println();

        //是否支持标记
        if (!inputStream.markSupported()) {
            System.out.println("该字节流不支持标记");
        } else {
            System.out.println("该字节流支持标记");
        }

        //标记, 已经读取5个字节,标记处为0x66
        System.out.println("标记该字节流为位置为0x66(f)");
        inputStream.mark(0);

        //跳过2个字节
        inputStream.skip(2);

        //读取5个字节到buffer
        byte [] buffer = new byte[5];
        inputStream.read(buffer, 0, 5);
        System.out.println("buffer: " + new String(buffer));

        //重置
        inputStream.reset();
        inputStream.read(buffer, 0, 5);
        System.out.println("重置后读取5个字符为: " + new String(buffer));
    }
}
```