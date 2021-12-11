# 简介

1. 网络数据的基本单位总是字节，java NIO提供ByteBuffer作为字节的容器，但是ByteBuffer使用起来过于复杂和繁琐。

2. ByteBuf是netty的Server与Client之间通信的数据传输载体(Netty的数据容器)，它提供了一个byte数组(byte[])的抽象视图，既解决了JDK API的局限性，又为网络应用程序的开发者提供了更好的API

3. ByteBuffer缺点
   - ByteBuffer长度固定，一旦分配完成，它的容量不能动态扩展和收缩，当需要编码的POJO对象大于ByteBuffer的容量时，会发生索引越界异常；
   - ByteBuffer只有一个标识位置的指针position，读写的时候需要手工调用flip()和rewind()等，使用者必须小心谨慎地处理这些API，否则很容易导致程序处理失败；
   - ByteBuffer的API功能有限，一些高级和实用的特性它不支持，需要使用者自己编程实现。
   
5.  ByteBuf优点
   - 容量可以按需增长
   - 读写模式切换不需要调用flip()
   - 读写使用了不同的索引
   - 支持方法的链式调用
   - 支持引用计数
   - 支持池化
   - 可以被用户自定义的缓冲区类型扩展
   - 通过内置的复合缓冲区类型实现透明的零拷贝

参考博客

```
https://blog.csdn.net/usagoole/article/details/88024517
```

# API详解
### ByteBuf工作机制
1. ByteBuf维护了两个不同的索引，一个用于读取，一个用于写入。readerIndex和writerIndex的初始值都是0，当从ByteBuf中读取数据时，它的readerIndex将会被递增(它不会超过writerIndex)，当向ByteBuf写入数据时，它的writerIndex会递增。
2. 名称以readXXX或者writeXXX开头的ByteBuf方法，会推进对应的索引，而以setXXX或getXXX开头的操作不会。
3. 在读取之后，0～readerIndex的就被视为discard的，调用discardReadBytes方法，可以释放这部分空间，它的作用类似ByteBuffer的compact()方法。
4. readerIndex和writerIndex之间的数据是可读取的，等价于ByteBuffer的position和limit之间的数据。writerIndex和capacity之间的空间是可写的，等价于ByteBuffer的limit和capacity之间的可用空间。



### 索引变化图

1. 初始分配
```
    +-------------------------------+
    |       writable bytes          |
    +-------------------------------+
    |                               |
    0=readerIndex=writerIndex       capacity
```
2. 写入N个字节
```
    +------------------+-------------------+
    |  readable bytes  |    writable bytes |
    +------------------+-------------------+
    |                  |                   |
    0=readerIndex      N=writerIndex       capacity
```
3. 读取M（＜N）个字节之后
```
    +-------------------+------------------+------------------+
    | discardable bytes |  readable bytes  |  writable bytes  |
    +-------------------+------------------+------------------+
    |                   |                  |                  |
    0               M=readerIndex    N=writerIndex       capacity
```
4. 调用discardReadBytes操作之后
```
    +------------------+----------------------+
    |  readable bytes  |    writable bytes    |
    +------------------+----------------------+
    |                  |                      |
    0=readerIndex   N-M=writerIndex         capacity
```
5. 调用clear操作之后
```
    +-------------------------------+
    |       writable bytes          |
    +-------------------------------+
    |                               |
    0=readerIndex=writerIndex       capacity
```
### 创建
推荐通过一个Unpooled的工具类来创建新的buffer而不是通过构造器来创建

- 创建堆缓冲区
```java
@Test
public void testHeapByteBuf() {
    ByteBuf heapBuf = Unpooled.buffer(10);
    if (heapBuf.hasArray()) {
        byte[] array = heapBuf.array();
        int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();
        int length = heapBuf.readableBytes();
        //0,0        
        logger.info("offset:{},length:{}", offset, length);
    }
}
```
- 直接内存缓冲区
```java
@Test
public void testDirectByteBuf() {
    ByteBuf directBuffer = Unpooled.directBuffer(10);
    if (!directBuffer.hasArray()) {
        int length = directBuffer.readableBytes();
        byte[] array = new byte[length];
        ByteBuf bytes = directBuffer.getBytes(directBuffer.readerIndex(), array);
        //0,0
        logger.info("offset:{},length:{}",bytes.readerIndex() , array.length);
    }
}
```

### 访问
1. ByteBuf提供了两个指针变量来支持顺序读写操作readerIndex用来支持读操作, writerIndex用来支持写操作。下图展示了ByteBuf是如何被两个索引分成三个区域的。

   ![img](../../sources\img\15499409399511.jpg)

2. 可读字节
ByteBuf的可读字节分段存储了实际数据。
- 新分配的、包装的或者复制的缓冲区的默认的readerIndex值为0。
- 任何名称以read或者skip开头的操作都将检索或者跳过位于当前readerIndex的数据，并且将它增加已读字节数。
- 如果被调用的方法需要一个ByteBuf参数作为写入的目标，并且没有指定目标索引参数，那么该目标缓冲区的writeIndex也将增加(例如:readBytes(ByteBuf dst))
```java
//读取所有可读的字节
ByteBuf buffer = ...;
while (buffer.readable()) {
    System.out.println(buffer.readByte());
}
```
3. 可写字节
可写字节分段是指一个拥有未定义内容的、写人就绪的内存区域。 新分配的缓冲区的writerindex的默认值为0。 任何名称以write开头的操作都将从当前的 writerIndex处开始写数据，并将它增加已经写入的字节数。如果写操作的目标也是 ByteBuf时，并且没有指定源索引的值，则源缓冲区的readerIndex也同样会被增加相同的大小(例如:writeBytes(ByteBuf dest))。
```java
@Test
public void testWrite() {
    ByteBuf heapBuf = Unpooled.buffer(10);
    while (heapBuf.writableBytes() > 4) {
        heapBuf.writeInt(new Random().nextInt());
    }
}
```
4. 丢弃字节
可丢弃字节的分段包含了已经被读过的字节。通过调用discardReadBytes()方法，可以丢弃它们并回收空间。这个分段的初始大小为0，存储在readerIndex中， 会随着read操作的执行而增加 (get*操作不会移动readerindex )

**虽然你可能会倾向于频繁地调用discardReadBytes()方法以确保可写分段的最大化，但是请注意，这将极有可能会导致内存复制， 因为可读字节必须被移动到缓冲区的开始位置。我们建议只在有真正需要的时候才这样做，例如,当内存非常宝贵的时候**

5. 清除buffer索引
你可以通过调用clear()将readerIndex和writerIndex都设为0。这不会清除buffer内容(例如用0填充), 他仅仅是清除了两个指针。请注意这个操作的语义和ByteBuffer.clear()是不一样的。调用clear()比调用discardReadBytes()轻量的多，因为只是重置索引而不会复制内存
```java
@Test
public void testFind() {
    ByteBuf heapBuf = Unpooled.buffer(13);
    heapBuf.writeByte(new Random().nextInt());
    heapBuf.writeByte(new Random().nextInt());
    heapBuf.writeBytes("\r\n".getBytes());
    int i = heapBuf.indexOf(0, 12, (byte) '\r');
    //2
    System.out.println(i);
    i = heapBuf.forEachByte(ByteProcessor.FIND_CRLF);
    //2
    System.out.println(i);
}
```
6. 派生缓冲区为ByteBuf提供以专门的方式呈现其内容的视图。
```java
duplicate()
slice()
slice(int, int)
Unpooled.unmodifiableBuffer ()
order (ByteOrder)
readSlice (int)
```
   每个这些方法都将返回一个新的ByteBuf实例，它具有自己的读索引、写索引和标记
   索引。其内部存储和 JDK 的ByteBuffer一样也是共享的。这使得派生缓冲区的创建成本
   是很低廉的，但是这也意味着，如果你修改了它的内容，也同时修改了其对应的源实例，所以要小心。
7. ByteBuf复制
如果需要一个现有缓冲区的真实副本，请使用copy()或者copy(int,int)。不同于派生缓冲区，由这个调用所返回的ByteBuf拥有独立的数据副本 。
```java
@Test
public void testSlice() {
    ByteBuf byteBuf = Unpooled.copiedBuffer("jannal", Charset.forName("utf-8"));
    ByteBuf slice = byteBuf.slice(0, 6);
    //jannal
    System.out.println(slice.toString(Charset.forName("utf-8")));
    byteBuf.setByte(0, (byte) 'J');
    assert byteBuf.getByte(0) == slice.getByte(0);
    //J
    System.out.println((char)byteBuf.getByte(0));
    //J
    System.out.println((char)slice.getByte(0));

    //复制

    ByteBuf copy = byteBuf.copy(0, 6);
    System.out.println(copy.toString(Charset.forName("utf-8")));

    byteBuf.setByte(0, (byte) 'j');
    //j
    System.out.println((char)byteBuf.getByte(0));
    //J
    System.out.println((char)copy.getByte(0));
}    
```
10. 与原生字节数组一样，ByteBuf使用零索引开始，最后一个字节的索引是capacity-1遍历buffer的所有字节

```java
@Test
public void testRandomAccess() {
    ByteBuf buffer = Unpooled.buffer(10);
    for (int i = 0; i < buffer.capacity(); i++) {
        byte b = buffer.getByte(i);
        //输出:0000000000
        System.out.print(b);
    }
}
```

### API方法
读写操作

- get()和 set()操作，从给定的索引开始,并且保持索引不变
- read()和 write()操作，从给定的索引开始，并且会根据已经访问过的字节数对索引进行调整 。

| 方法 | 	描述|
| ---- | ---- | 
|  setBoolean (int , boolean)    |    设定给定索引处的 Boolean 值  |
|getBoolean(int)	|返回给定索引处的 Boolean 值|
|setByte(int index, int value)|	设定给定索引处的字节值|
|getByte(int)	|返回给定索引处的字节|
|getUnsignedByte(int )	|将给定索引处的无符号字节值作为 short 返回|
|setMedium(int index , int value)	|设定给定索引处的 24 位的中等 int值|
|getMedium(int)	|返回给定索引处的 24 位的中等 int 值|
|getUnsignedMedium (int)	|返回给定索引处的无符号的 24 位的中等 int 值|
|setint(int index , int value)|	设定给定索引处的 int 值|
|getint (int)|	返回给定索引处的 int 值|
|getUnsignedint(int)	|将给定索引处的无符号 int 值作为 long 返回|
|setLong(int index, long value)|	设定给定索引处的 long 值|
|getLong(int)|	返回给定索引处的 long 值|
|setShort(int index, int value)|	设定给定索引处的 short 值|
|getShort(int)	|返回给定索引处的 short 值|
|getUnsignedShort(int)	|将给定索引处的无符号 short 值作为 int 返回|
|getBytes (int, …)	|将该缓冲区中从给定索引开始的数据传送到指定的目的地|

```java
@Test
public void testGetSet(){
    ByteBuf byteBuf = Unpooled.copiedBuffer("jannal", Charset.forName("utf-8"));
    System.out.println((char)byteBuf.getByte(0));
    int readIndex = byteBuf.readerIndex();
    int writerIndex= byteBuf.writerIndex();
    byteBuf.setByte(0,(byte)'J');
    System.out.println((char)byteBuf.getByte(0));
    assert readIndex== byteBuf.readerIndex();
    assert writerIndex== byteBuf.writerIndex();
}

@Test
public void testReadWrite(){
    ByteBuf byteBuf = Unpooled.copiedBuffer("jannal", Charset.forName("utf-8"));
    System.out.println((char)byteBuf.readByte());
    int readIndex = byteBuf.readerIndex();
    int writerIndex= byteBuf.writerIndex();
    byteBuf.writeByte((byte)'J');
    assert readIndex== byteBuf.readerIndex();
    assert writerIndex!= byteBuf.writerIndex();
}
```



其他操作
|方法	|描述|
|---------|----------|
|isReadable ()	|如果至少有一个字节可供读取，则返回 true|
|isWritable ()|	如果至少有一个字节可被写入，则返回 true|
|readableBytes()|	返回可被读取的字节数|
|writableBytes()	|返回可被写入的字节数|
|capacity()	|返回 ByteBuf 可容纳的字节数 。在此之后，它会尝试再次扩展直到达到maxCapacity ()|
|maxCapacity()	|返问 ByteBuf 可以容纳的最大字节数|
|hasArray()	|如果 ByteBuf 由一个字节数组支撑，则返回 true|
|array ()	|如果 ByteBuf 由一个字节数组支撑则返问该数组;否则，它将抛出 一个 UnsupportedOperat工onException 异常|


### 辅助类
#### CompositeByteBuf
CompositeByteBuf允许将多个ByteBuf的实例组装到一起，形成一个统一的视图，有点类似于数据库将多个表的字段组装到一起统一用视图展示。


#### CompositeByteBuf
在一些场景下非常有用，例如某个协议POJO对象包含两部分：消息头和消息体，它们都是ByteBuf对象。当需要对消息进行编码的时候需要进行整合，如果使用JDK的默认能力，有以下两种方式：

- 将某个ByteBuffer复制到另一个ByteBuffer中，或者创建一个新的ByteBuffer，将两者复制到新建的ByteBuffer中。这样会导致两次数据复制的操作
```java
ByteBuf allBuf = Unpooled.buffer(header.readableBytes() + body.readableBytes());
allBuf.writeBytes(header);
allBuf.writeBytes(body);
```
- 通过List或数组等容器，将消息头和消息体放到容器中进行统一维护和处理
##### 更好的实现方式(零拷贝)
```java
ByteBuf header = Unpooled.buffer(10);
ByteBuf body = Unpooled.buffer(10);
CompositeByteBuf compositeByteBuf = Unpooled.compositeBuffer();
//其中第一个参数是 true, 表示当添加新的 ByteBuf 时, 自动递增
//CompositeByteBuf 的 writeIndex
compositeByteBuf.addComponents(true, header, body);
```

将header和body合并为一个逻辑上的ByteBuf，如下图.在 CompositeByteBuf 内部, 这两个 ByteBuf 都是单独存在的, CompositeByteBuf 只是逻辑上是一个整体

![img](../../sources\img\15500620111851.jpg)

```java
ByteBuf header = Unpooled.buffer(10);
ByteBuf body = Unpooled.buffer(10);
//Unpooled.wrappedBuffer 方法, 它底层封装了 CompositeByteBuf 操作
ByteBuf allByteBuf = Unpooled.wrappedBuffer(header, body);
```
#### ByteBufHolder
1. ByteBufHolder是ByteBuf的容器，在Netty中，它非常有用，例如HTTP协议的请求消息和应答消息都可以携带消息体，这个消息体在NIO ByteBuffer中就是个ByteBuffer对象，在Netty中就是ByteBuf对象。由于不同的协议消息体可以包含不同的协议字段和功能，因此，需要对ByteBuf进行包装和抽象，不同的子类可以有不同的实现。为了满足这些定制化的需求，Netty抽象出了ByteBufHolder对象，它包含了一个ByteBuf，另外还提供了一些其他实用的方法，使用者继承ByteBufHolder接口后可以按需封装自己的实现。

ByteBufHolder继承了ReferenceCounted

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| content()                                                    | 返回由这个 ByteBufHolder 所持有的 ByteBuf                    |
| copy( )                                                      | 返问这个 ByteBufHolder 的一个深拷贝，包括一个其所包含的 ByteBuf 的非共享拷贝 |
| duplicate()                                                  | 返回这个 ByteBufHolder 的一个浅拷贝，包括一个其所包含的 ByteBuf 的共享拷贝 |


#### ByteBufAllocator                                             
1. 为了降低分配和释放内存的开销， Netty通过ByteBufAllocator实现了 (ByteBuf 的)池化，它可以用来分配我们所描述过的任意类型的 ByteBuf 实例                                                

2. 方法
| 方法 |	描述 |
|---------|---------------------------|
|buffer()   buffer(int initialCapacity);   buffer(int initialCapacity, int maxCapacity);|	返回一个基于堆或者直接内存存储的 ByteBuf|
|heapBuffer ()    heapBuffer(int initialCapacity)   heapBuffer(int initialCapacity, int maxCapacity)	|返回一个基于堆内存存储的 ByteBuf|
|directBuffer()   directBuffer(int initialCapacity)  directBuffer(int initialCapacity , int maxCapacity)	|返回一个基于直接内存存储的 ByteBuf|
|compositeBuffer()compositeBuffer(int maxNumComponents) compositeDirectBuffer()compositeDirectBuffer (int maxNumComponents); compositeHeapBuffer()compositeHeapBuffer(int maxNumComponents);	|返回一个可以通过添加最大到指定数目的基于堆的或者直接内存存储的缓冲区来扩展的 CompositeByteBuf|
|ioBuffer()|	返回一个用于套接字的 I/O 操作的 ByteBuf。默认地， 当所运行的环境具有 sun.misc.Unsafe支持时，返回基于直接内存存储的 ByteBuf，否则返回基于堆内存存储的 ByteBuf;当指定使用 PreferHeapByteBufAllocator 时，则只会返回基于堆内存存储的 ByteBuf|

3. 可以通过Channel或者绑定到ChannelHandler的ChannelHandlerContext获取一个到ByteBufAllocator的引用。

   Netty提供了两种ByteBufAllocator的实现，PooledByteBufAllocator和UnpooledByteBufAllocator。

   - 前者池化了ByteBuf的实例以提高性能并最大限度地减少内存碎片,此实现使用了一种称为jemalloc的已被大量现代操作系统所采用的高效方法来分配内存。 
   - 后者的实现不池化 ByteBuf实例，并且在每次它被调用时都会返回一个新的实例 。

```java
Channel channel = ...;
ByteBufAllocator allocator = channel.alloc();
...

    ChannelHandlerContext ctx = ...;
ByteBufAllocator allocator2 = ctx.alloc()
```
#### Unpooled
1. Unpooled提供了静态的辅助方法来创建未池化的ByteBuf实例。Unpooled类还使得 ByteBuf同样可用于那些并不需要Netty的其他组件的非网络项目，使得其能得益于高性能的可扩展的缓冲区API。

|方法	|描述|
|------------|--------------------------------|
|buffer()buffer(int 工nitialCapacity)buffer(int initialCapacity, int maxCapacity)	|返回一个未池化的基于堆内存存储的ByteBuf|
|directBuffer()directBuffer(int initialCapacity)directBuffer(int initialCapacity, int maxCapacity)	|返回一个未池化的基于直接内存存储ByteBuf|
|wrappedBuffer()	|返回一个包装了给定数据的ByteBuf|
|copiedBuffer()	|返回一个复制了给定数据的 ByteBuf|


##### ByteBufUtil
1. ByteBufUtil提供了用于操作ByteBuf的静态的辅助方法。因为这个API是通用的，并且和池化无关，所以这些方法已然在分配类的外部实现 。
2. 这些静态方法中最有价值的可能就是hexdump()方法，它以十六进制的表示形式打印 ByteBuf 的内容。 这在各种情况下都很有用，例如，出于调试 的目的记录ByteBuf的内容。十六进制的表示通常会提供一个比字节值的直接表示形式更加有用的日志条目，此外，十六进制的版本还可以很容易地转换回实际的字节表示 。
3. 另一个有用的方法是boolean equals(ByteBuf , ByteBuf)，它被用来判断两个ByteBuf实例的相等性。 如果你实现自己的 ByteBuf子类，你可能会发现ByteBufUtil的其他有用方法。
4. encodeString(ByteBufAllocator alloc, CharBuffer src, Charset charset):对需要编码的字符串src按照指定的字符集charset进行编码，利用指定的ByteBufAllocator生成一个新的ByteBuf。
5. decodeString(ByteBuffer src, Charset charset)：使用指定的ByteBuffer和charset进行对ByteBuffer进行解码，获取解码后的字符串。
##### 零拷贝
1. byte数组转换为ByteBuf对象。Unpooled.wrappedBuffer方法来将bytes 包装成为一个UnpooledHeapByteBuf对象, 而在包装的过程中, 是不会有拷贝操作的。最后我们生成的生成的ByteBuf对象是和bytes数组共用了同一个存储空间, 对bytes的修改也会反映到ByteBuf对象中.

```java
1. 传统做法，这样的方式也是有一个额外的拷贝操作的
byte[] bytes = ...
ByteBuf byteBuf = Unpooled.buffer();
byteBuf.writeBytes(bytes);

2. 无额外copy方式
byte[] bytes = ...
ByteBuf byteBuf = Unpooled.wrappedBuffer(bytes);
```

2. Unpooled工具类还提供了很多重载的wrappedBuffer方法:这些方法可以将一个或多个buffer包装为一个ByteBuf对象, 从而避免了拷贝操作。

3. 通过slice操作实现零拷贝: slice操作和wrap操作刚好相反, Unpooled.wrappedBuffer可以将多个ByteBuf合并为一个, 而slice操作可以将一个ByteBuf切片 为多个共享一个存储区域的ByteBuf对象。

4. ByteBuf 提供了两个 slice 操作方法:

```java
public ByteBuf slice();
public ByteBuf slice(int index, int length);
```


5. 不带参数的slice方法等同于buf.slice(buf.readerIndex(), buf.readableBytes())调用, 即返回buf中可读部分的切片. 而 slice(int index, int length) 方法相对就比较灵活了, 我们可以设置不同的参数来获取到 buf的不同区域的切片.

    ByteBuf byteBuf = ...
    ByteBuf header = byteBuf.slice(0, 5);
    ByteBuf body = byteBuf.slice(5, 10);

6. 用slice方法产生header和body的过程是没有拷贝操作的,header和 body对象在内部其实是共享了byteBuf存储空间的不同部分而已. 即:

   ![img](../../sources\img\15500631630964.jpg)

7. 通过FileRegion实现零拷贝:Netty中使用FileRegion实现文件传输的零拷贝, 不过在底层FileRegion是依赖于 Java NIOFile Channel.transfer的零拷贝功能.

8. Netty 的 Zero-copy 体现在如下几个个方面:

- Netty 提供了CompositeByteBuf类, 它可以将多个ByteBuf合并为一个逻辑上的ByteBuf, 避免了各个ByteBuf之间的拷贝.
- 通过wrap操作, 我们可以将byte[]数组、ByteBuf、ByteBuffer等包装成一个 Netty的ByteBuf对象, 进而避免了拷贝操作.
- ByteBuf支持slice操作, 因此可以将ByteBuf分解为多个共享同一个存储区域的 ByteBuf, 避免了内存的拷贝.
- 通过FileRegion包装的FileChannel.tranferTo实现文件传输, 可以直接将文件缓冲区的数据发送到目标Channel, 避免了传统通过循环write方式导致的内存拷贝问题.
9. Netty 中的 Zero-copy与上面我们所提到到 OS 层面上的 Zero-copy 不太一样, Netty的 Zero-copy 完全是在用户态(Java 层面)的, 它的 Zero-copy的更多的是偏向于优化数据操作 这样的概念.

#### 转化为已存在的JDK类型
##### Byte Array
1. 假如一个ByteBuf是有一个byte数组作为支持的, 你可以直接通过array()方法访问它。判断一个buffer是否是被byte array作为支持,调用hasArray()
2. 只有堆内内存的ByteBuf是有array支持的, 如果是堆外内存的ByteBuf, 是不能通过array()获取到数据的, 而CompositeByteBuf可能由堆内的ByteBuf和堆外的DirectByteBuf组成, 所以它也不能直接通过array()获取数据
##### NIO Buffers
1. 如果一个ByteBuf可以被转换为NIO ByteBuffer,它共享它的内容,你可以通过nioBuffer()获取它。判断一个buffer能否被转化为NIO buffer, 使用nioBufferCount().

2. Strings
各种各样的toString(Charset)方法将一个ByteBuf转化为一个String.请注意toString()并不是一个转换方法.

3. I/O Streams
请看ByteBufInputStream和ByteBufOutputStream
