# 简介
主要内容包括OutputStream及其部分子类，以分析源代码的方式学习。关心的问题包括：
- 每个字节输出流的作用
- 各个流之间的主要区别
- 何时使用某个流
- 区分节点流和处理流
- 流的输出目标等问题。
 OutputStream的类树如下所示，其中，ObjectOutputStream和PipedOutputStream本文将不做讨论。
```
java.io.OutputStream (implements java.io.Closeable, java.io.Flushable)
    java.io.ByteArrayOutputStream
    java.io.FileOutputStream
    java.io.FilterOutputStream
        java.io.BufferedOutputStream
        java.io.DataOutputStream (implements java.io.DataOutput)
        java.io.PrintStream (implements java.lang.Appendable, java.io.Closeable)
    java.io.ObjectOutputStream (implements java.io.ObjectOutput, java.io.ObjectStreamConstants)
    java.io.PipedOutputStream
```


# OutputStream 
```
package java.io;

//它是抽象类，并且实现了两个接口Closeable和Flushable。
public abstract class OutputStream implements Closeable, Flushable {

    
    public abstract void write(int b) throws IOException;


    public void write(byte b[]) throws IOException {
        write(b, 0, b.length);
    }

    public void write(byte b[], int off, int len) throws IOException {
        //数组不能为空，否则抛出NullPointerException
        if (b == null) {
            throw new NullPointerException();
        } else if ((off < 0) || (off > b.length) || (len < 0) ||
                   ((off + len) > b.length) || ((off + len) < 0)) {
              //此处判断off+len<0是多余的
              throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return;
        }
        //最终会调用第一个write方法。注意：1.子类可能会复写当前的write方法；2.在输出的过程中，还是一个一个字节输出的。
        for (int i = 0 ; i < len ; i++) {
            write(b[off + i]);
        }
    }

    public void flush() throws IOException { }
    public void close() throws IOException { }

}
```

# 方法介绍

## write(int b)
1. 作为抽象类中唯一的抽象方法，（非抽象）子类必须实现这个方法。`write(byte b[])`和`write(byte b[], int off, int len)`方法也是调用这个类来完成的。

2. 对于一个输出流，我们需要关心输出的内容到哪里去了，从这个write方法中我们根本看不到输出的目的地，所以实现这个方法的子类必须告诉这一点，而实现这个方法的子类，就是节点流。
***  注意  ***
作为字节输出流，为何这里参数传递为int型，而非byte型。

## write(byte b[])
此方法直接输出一个字节数组中的全部内容，调用了下面的write方法。

## write(byte b[], int off, int len)
要输出的内容已存储在了字节数组b[]中，但并非全部输出，只输出从数组off位置开始的len个字节。因此，需要对传入的三个参数作合理性判断。

## flush()和close()
这两个方法就是实现两个接口时分别需要实现的方法，但这里方法中内容是空的，子类可以override这两个方法，如果子类不复写，则此方法为空。 
