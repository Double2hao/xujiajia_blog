#由LevelDB理解 LSM-Tree
>  
 本文参考 《大数据日知录》  


# 概念

LSM-Tree（Log-structured Merge tree）是一种数据结构，它的本质是将大量的随机写操作转换成批量的序列写。 LSM-Tree可以极大提升磁盘数据写入的速度，所以LSM-Tree非常适合对写操作效率有高要求的应用场景。

## 为何“随机写”改成“序列写”可以提高磁盘写入速度

<img src="https://img-blog.csdnimg.cn/20190223100948837.png" alt="在这里插入图片描述"> 磁盘每次读写数据，不管你读写的数据有多小，都是会读写一个扇区的数据。

如果使用随机写的方式，每次都需要至少一次的“写扇区”操作。那么如果随机写 n个数据，就需要执行n次“写扇区”的操作。 改成“序列写”之后，只要这n个数据的大小没有超过一个扇区，那么只需要执行一次的“写扇区”操作。

>  
 为什么随机写不能通过一次写多个来提高效率？ 也是可以的，但是随机写的数据不是顺序存储的，这就意味着还是需要读写多个扇区，所以即使一次写多个，提升的性能也不能与“序列写”相比。 


# LevelDB静态结构

LevelDB的LSM-Tree结构以及管理过程，是一种非常典型的LSM-Tree的实现，因此本文直接以LevelDB的结构为例来讲解LSM-Tree。 <img src="https://img-blog.csdnimg.cn/20190511141528547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

## 写过程
1. 写入log文件。（防止意外情况）1. 写入内存中的MemTable1. 内存中的MemTable大小到达一定程度后，原先的MemTable就变成了Immutable MemTable，并它只可读，不可写。LevelDB后台调用会将Immutable MemTable中的数据存储到磁盘中。1. 内存中创建一个新的MemTable，后续的数据写入新的这个MemTable。
# LevelDB写入磁盘过程

由上图可知，硬盘中存储的是多个.sst文件，即SSTable（Sorted String Table），SSTable的结构没有什么特别，直接把它当做一个顺序存储的数据结构就可以。

整个操作其实非常简单，直接将Immutable MemTable中的数据存储到多个SSTable，然后将这些SSTable存储到整个SStable数组的头部（也就是Level 0）就可以了。

>  
 假设此时硬盘中已经有了两个SSTable，此时硬盘中的内容为： level 0-&gt;tableA level 1-&gt;tableB 
 此时要插入一个tableC。 那么插入后的硬盘中的数据结构为： level 0-&gt;tableC level 1-&gt;tableA level 2-&gt;tableB 


## 读过程

由上面的写入过程可以了解到，级别较低的SSTable中的内容是更新的。这时候再来看LevelDB的读过程就比较好理解了。
1. 先去MemTable中读1. 如果没有读到就去Immutable MemTable中读1. 如果还是没有读到就到硬盘中去读，读的顺序是level从低到高。
<img src="https://img-blog.csdnimg.cn/20190511144419947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# 读过程举例

>  
 假设此刻硬盘中有两个SSTable，结构如下： level 0-&gt;tableA level 1-&gt;tableB tableA中存储的值为：{“a”:123,“c”:321} tableB中存储的值为：{“b”:222,“c”:333} 


## 例子一

>  
 这时候要读取“a”的value，假设内存中没有读到这个值，那么硬盘中读取的顺序应该是先读取level 0，再读取level 1。 读取level 0的时候直接可以读取到"a"的value为123，于是就返回123。 


## 例子二

>  
 这时候要读取“b”的value，假设内存中没有读到这个值，那么硬盘中读取的顺序应该是先读取level 0，再读取level 1。 先读取level 0，发现tableA中并没有“b”这个key的value，于是到level 1中去寻找，在tableB中发现“b”的value为222，于是就返回222。 


# 不同Level的SSTable合并

levelDB会轮流让level高的文件去合并level低的文件的内容，合并完成之后，level高的文件被替换成合并后的文件，level低的文件则被删除。

>  
 比如level 0中有A.sst，level 1中有A.sst，两个文件合并后生成A_new.sst， level 1中的A.sst替换成 A_new.sst，level 0 的A.sst则删除。 


由于.sst中的内容本身就是有序的，因此合并的过程就非常简单，使用多路归并排序的方式，每次将两个旧文件中最小的key记录，最终就能得到一个新的有序的SSTable，再存储为.sst文件。
