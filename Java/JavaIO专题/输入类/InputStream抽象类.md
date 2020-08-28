# 简介
InputStream这个抽象类是所有基于字节的输入流的超类，抽象了Java的字节输入模型。在这个类中定义了一些基本的方法。
```
public abstract class InputStream implements Closeable
```
首先这是一个抽象类，实现了Closeable接口，也Closeable接口又拓展了AutoCloseable接口，因此所有InputStream及其子类都可以用于Java 7 新引入的带资源的try语句。读入字节之前，我们可能想要先知道还有多少数据可用，这有available方法完成，具体的读入由read（）及其重载方法完成，skip方法用于跳过某些字节，同时定义了几个有关标记（mark）的方法，读完数据使用close方法关闭流，释放资源。下面详细介绍各个方 法：

##  available方法
```
public int available() throws IOException
```
假设方法返回的int值为a，a代表的是在不阻塞的情况下，可以读入或者跳过（skip）的字节数。也就是说，在该对象下一次调用读入方法读入a个字节，或者skip方法跳过a个字节时，不会出现阻塞（block）的情况。
这个调用可以由相同线程调用，也可以是其他线程调用。但是在下次读入或跳过的时候，实际读入（跳过）的可能不只a字节。当遇到流结尾的时候，返回0。如果出现I/O错误，抛出IOException异常。看一下InputStream中该方法的实现：
```
public int available() throws IOException { 
      return 0; 
} 
```
只是简单地返回0，因此子类必须重写该方法。注意到一点，虽然这个方法实现中根本不会出现异常，但是还是在throws中指出（specify）可能抛出 IOException。这是Java异常机制很重要的一个点，子类的方法不能throws父类方法没有throws的异常（构造器除外），因此在父类方法先指出，然后允许子类方法抛出IOException。 单独使用这一方法几乎没有意义，它一般用于在读入或者跳过之间先探测一下有多少可用字节。

## 读入方法：read
跟读入相关的方法是这个类的核心方法。有3种重载的形式，下面分别介绍。
　　
### read()
```
public abstract int read() throws IOException
```
　　读取输入流的下一个字节。这是一个抽象方法，不提供实现，子类必须实现这个方法。该方法读取下一个字节，返回一个0-255之间的int类型整数。如果到达流的末端，返回-1. 
调用该方法的时候，方法阻塞直到出现下列其中一种情况：
- 遇到流的尾部（end of the stream）。
- 有数据可以读入。
- 抛出异常。 
面向字节的操作时，可能需要像这样比较底层的字节操作。我们也可以一次读入多个字节，使用下面的重载形式。

### read（byte[] b)
```
public int read(byte b[]) throws IOException
```
　　试图读入多个字节，存入字节数组b，返回实际读入的字节数。如果传递的是一个空数组（注意数组长度可以为0，即空数组。比如 byte[] b = new byte[0]; 或者byte[] b = {};）那么什么也没读入，返回0.
　　如果到达流尾部，没有字节可读，返回-1；如果上面两种情况都没有出现，并且没有I/O错误，则至少有1个字节被读入，存储到字节数组b中。实际读入的第一个字节存在b[0]，往后一次存入数组，读入的字节数最多不能超过数组b的长度。如果读入的字节数小于b的长度，剩余的数组元素保持不变。具体地，如果读入的字节数为k，则k个字节分别存在 b[0]到b[k-1]，而b[k]到b[b.length-1]保持原来的数据。
　　
###  read (byte[] b, int off, int len)
```
public int read(byte[] b,int off,int len) throws IOException
```
　　这个方法跟上一个功能类似，除了读入的数据存储到b数组是从off开始。len是试图读入的字节数，返回的是实际读入的字节数。如果len=0，则什么也不读入，返回0；如果遇到流尾部，返回-1.否则至少读入一个字节。
假设实际读入k个字节，则k个字节分别存储在b[off]到b[off+k-1]，而b[off+k]往后的元素保持不变。b[off]之前也保持不变。
```
public int read(byte b[]) throws IOException { 
    return read(b, 0, b.length); 
}
```
　　解析来看一下第三个read方法的源代码：
```
public int read(byte b[], int off, int len) throws IOException {
        if (b == null) {   // 检测参数是否为null
            throw new NullPointerException();
        } else if (off < 0 || len < 0 || len > b.length - off) {
            throw new IndexOutOfBoundsException(); // 数组越界检测
        } else if (len == 0) {
            return 0;   //如果b为空数组，返回0
        }

        int c = read(); // 调用read（）方法获取下一个字节
        if (c == -1) {
            return -1;
        }               // 遇到流尾部，返回-1
        b[off] = (byte)c;  //读入的第一个字节存入b[off]

        int i = 1;    // 统计实际读入的字节数
        try {
            for (; i < len ; i++) { // 循环调用read，直到流尾部
                c = read();
                if (c == -1) {
                    break;
                }
                b[off + i] = (byte)c; // 一次存入字节数组
            }
        } catch (IOException ee) {
        }
        return i;  // 返回实际读入的字节数
    }
```
　　我们看到方法可能抛出IOException异常，如果第一个字节无法读入且原因不是到达流尾部，或者流已经被关闭，或者其他IO错误，则抛出这个异常。
　　上面三个读入方法都可能出现阻塞。理解这三个方法很重要的一点是：方法只是尝试读入我们想要的字节数，但是能否成功， 会受到数据源的影响。另外一点就是读入的数据存到哪里，第一个方法作为返回值，第二、三个方法存入到指定数组的指定位置，返回的是实际读入的字节数。后两个方法真正的读入工作都是通过调用抽象方法read（）来完成的，资格抽象方法在子类中实现。
　　
### skip方法
```
public long skip(long n)throws IOException 
```
这个方法试图跳过当前流的n个字节，返回实际跳过的字节数。如果n为负数，返回0.当然子类可能提供不能的处理方式。n只是我们的期望，至于具体跳过几个，则不受我们控制，比如遇到流结尾。
```
public long skip(long n) throws IOException {
        long remaining = n; // 还有多少字节没跳过
        int nr;
        if (n <= 0) {
            return 0;  // n小于0 简单返回0
        }
        int size = (int)Math.min(MAX_SKIP_BUFFER_SIZE, remaining); // 这里的常数在类中定义为2048
        byte[] skipBuffer = new byte[size]; // 新建一个字节数组，如果n<2048，数组大小为n，否则为2048
        while (remaining > 0) {
            nr = read(skipBuffer, 0, (int)Math.min(size, remaining)); // 读入字节，存入数组 
            if (nr < 0) {  // 遇到流尾部跳出循环
                break;
            }
            remaining -= nr;
        }
        return n - remaining;
}
```
从代码的逻辑上可以看出，是通过不断地读取字节来完成跳过的任务的。首先建立一个缓冲数组，这个数组的大小不超过2048.如果要跳过的字节大于2048， 则数组大小为2048，否则采用要跳过的字节数作为数组长度。接下来不断读取字节，填入数组，如果还没跳过的字节数超过缓冲数组长度，则读入2048，否 则读入还没跳过的字节，完成跳过任务。如果遇到流尾部，跳出循环，返回已经读入的字节个数.


## 与标记相关的方法
与标注相关的方法使得我们可以重新读入（跳过）那些已经被我们读入或者跳过的字节，再重新来过一次。
### mark
```
public void mark(int readlimit) 
```
　　这个方法用于在流的当前位置做个标记，参数readLimit指定这个标记的“有效期“，如果从标记处开始往后，已经获取或者跳过了readLimit个字节，那么这个标记失效，不允许再重新回到这个位置（通过reset方法）。也就是你想回头不能走得太远呀，浪子回头不一定是岸了，跳过（获取）了太多字 节，标记就不再等你啦。多次调用这个方法，前面的标记会被覆盖。
4.2 reset
```
public void reset()throws IOException 
```
　　这个方法用于重定位到最近的标记。如果在这之前mark方法从来没被调用，或者标记已经无效，在抛出IOException。如果没有抛出这个异常，将当前位置重新定位到最近的标记位置。
```
InputStream is = null;
try {
    is = new BufferedInputStream(new FileInputStream("test.txt"));
    is.mark(4);
    is.skip(2);
    is.reset();<br>　　 System.out.println((char)is.read());         
} finally {
    if (is != null) {
        is.close();
    }
}
```
　　我们使用了支持mark的BufferedInputStream，首先一开始做标记，跳过两个自己，然后再回到最初的位置。
### markSupported
```
public boolean markSupported()
```
　　检测当前流对象是否支持标记。是返回true。否则返回false。比如InputStream不支持标记，而BufferedInputStream支持。

## close方法
```
public void close()throws IOException 
```
　　关闭当前流，释放与该流相关的资源，防止资源/泄露。在带资源的try语句中将被自动调用。关闭流之后还试图读取字节，会出现IOException异常。