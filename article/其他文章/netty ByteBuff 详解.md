#netty ByteBuff 详解
>  
 本文参考： 


# 概要

netty的ByteBuff是用来替代nio的ByteBuffer的，与nio的ByteBuffer相比有诸多优势，主要有以下几点：
1.  扩容问题 ByteBuffer长度固定，分多了会浪费内存，分少了会数组越界。如果发现存储的数据大于ByteBuffer的长度，那么就需要创建一个新的ByteBuffer，然后把旧的数据放到新的更大的ByteBuffer中。 ByteBuff如果需要存储大的数据会自动扩展。 1.  读写问题 ByteBuffer读写切换的时候需要手动调用flip()方法，使用不方便，提高了开发门槛。 ByteBuff读写都有对应的方法，一般情况下，不需要自己操作数据结构，使用方便。 1.  有多种ByteBuff类型针对不同场景优化。 1.  netty使用对象池提高了内存的使用率，减少每次新建对象的时间。 
# ByteBuff扩容

ByteBuff扩容最终会走到calculateNewCapacity()方法中，主要过程如下：
1. ByteBuff.writeBytes()1. ByteBuff.ensureWritable() 这个方法中会根据写入的内容计算出至少需要多少字节才可存储。1. ByteBuff.calculateNewCapacity() 这个方法中会根据自己的逻辑计算出最终新的ByteBuff的大小，逻辑如下： 1、如果新的容量大于4MB，则容量在原来基础上增加4MB。 2、如果原来容量小于4MB，大于64字节，那么容量翻倍。 3、如果容量小于64字节，那么就返回64字节。 4、ByteBuff最大容量可以自己设置，如果最终计算出的容量超过了设置的最大容量，那么就返回最大容量。
```
    public ByteBuf ensureWritable(int minWritableBytes) {
        if (minWritableBytes &lt; 0) {
            throw new IllegalArgumentException(String.format(
                    "minWritableBytes: %d (expected: &gt;= 0)", minWritableBytes));
        }

        if (minWritableBytes &lt;= writableBytes()) {
            return this;
        }

        if (minWritableBytes &gt; maxCapacity - writerIndex) {
            throw new IndexOutOfBoundsException(String.format(
                    "writerIndex(%d) + minWritableBytes(%d) exceeds maxCapacity(%d): %s",
                    writerIndex, minWritableBytes, maxCapacity, this));
        }

        // Normalize the current capacity to the power of 2.
        int newCapacity = calculateNewCapacity(writerIndex + minWritableBytes);

        // Adjust to the new capacity.
        capacity(newCapacity);
        return this;
    }

```

```
    private int calculateNewCapacity(int minNewCapacity) {
        final int maxCapacity = this.maxCapacity;
        final int threshold = 1048576 * 4; // 4 MiB page

        if (minNewCapacity == threshold) {
            return threshold;
        }

        // If over threshold, do not double but just increase by threshold.
        if (minNewCapacity &gt; threshold) {
            int newCapacity = minNewCapacity / threshold * threshold;
            if (newCapacity &gt; maxCapacity - threshold) {
                newCapacity = maxCapacity;
            } else {
                newCapacity += threshold;
            }
            return newCapacity;
        }

        // Not over threshold. Double up to 4 MiB, starting from 64.
        int newCapacity = 64;
        while (newCapacity &lt; minNewCapacity) {
            newCapacity &lt;&lt;= 1;
        }

        return Math.min(newCapacity, maxCapacity);
    }

```

# ByteBuff读写

<img src="https://img-blog.csdnimg.cn/20190713105948251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> ByteBuff被readerIndex和writeerIndex两个索引分成三部分： 1、可丢弃字节 2、可读字节 3、可写字节

具体内容根据各个方法作用分析比较容易理解，ByteBuff常用接口如下：

|方法|作用
|------
|isReadable()|如果writerIndex&gt;readerIndex则返回true
|isWritable()|如果capcity&gt;writeIndex则返回true
|readableBytes()|writerIndex-readerIndex ，返回可读字节容量
|writableBytes()|capacity-writerIndex ，返回可写字节容量
|discardReadBytes()|将可丢弃字节删除，可读字节和可写字节前移（每次调用会有复制逻辑，频繁调用会掉性能）
|clear()|设置readerIndex = writerIndex = 0,由于写入会直接从writerIndex开始写，因此相当于逻辑删除

# ByteBuff类型

ByteBuff有三种类型：
1. 堆内存缓冲区（HeapByteBuf）1. 直接内存缓冲区（DirectByteBuf）1. 复合缓冲区（CompositeByteBuf）
### 堆内存缓冲区（HeapByteBuf）

数据存储在堆中，可以认为就是我们常用的内存缓冲区，用来代替 nio的ByteBuffer。 与直接内存缓冲区相比，在使用网卡传输数据的时候会更慢一些，因为需要先把数据从内存缓冲区拷贝到内核缓冲区，然后内核缓冲区再使用网卡发送数据。

### 直接内存缓冲区（DirectByteBuf）

数据存储在方法区（在内核中）。 由于数据本身就存储在内核中，因此使用网卡传输数据的时候直接可以传输，不需要多余的拷贝。因此，这也被称为零拷贝。

>  
 从硬盘中读取数据使用网卡发送出去，一般步骤如下： 
 - 数据从磁盘读取到内核的read buffer- 数据从内核缓冲区拷贝到用户缓冲区- 数据从用户缓冲区拷贝到内核的socket buffer- 数据从内核的socket buffer拷贝到网卡接口（硬件）的缓冲区 
 使用内存缓冲区只需要两步： 
 - 调用transferTo,数据从文件由DMA引擎拷贝到内核read buffer- 接着DMA从内核read buffer将数据拷贝到网卡接口buffer 


### 复合缓冲区（CompositeByteBuf）

拿HTTP协议举例。我们使用HTTP协议的时候，经常header部分不会变动，只会变动主体部分。 <img src="https://img-blog.csdnimg.cn/20190713120122289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> 一般情况下，我们需要将头部部分拷贝到每次一次发送的ByteBuf中。

如果使用复合缓冲区的话，我们就不需要这么做。复合缓冲区可以将多个ByteBuff组合。 在这种情况下，我们可以将不会变动的HTTP的header单独设置一个ByteBuff，在主体部分的ByteBuff要发送的时候，使用复合缓冲区将两者组合起来一起发送。

这样一方面是逻辑解耦，另一方面是提高了数据的复用性。