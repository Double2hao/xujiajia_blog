#mysql索引原理：B-Tree 和 B+Tree简介
>  
 本文参考：https://www.kancloud.cn/kancloud/theory-of-mysql-index/41856 完全不了解B-Tree的读者可以先看下这篇文章： https://zhuanlan.zhihu.com/p/24309634 


# B-Tree

B-Tree毫无疑问是树结构，如下图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2410.png" alt="在这里插入图片描述"> 主要有以下特性：
- d为大于1的一个正整数，称为B-Tree的度。- 每个非叶子节点由n-1个key和n个指针组成，其中d&lt;=n&lt;=2d。- 所有叶节点具有相同的深度，等于树高h。
B-Tree是一个非常有效率的索引数据结构，如果一共查找的索引有N个，B-Tree的度为d，则查找的复杂度为O(logdN)。

# B+Tree

B+Tree是B-Tree的变种，如下图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2411.png" alt="在这里插入图片描述"> 各个特性和B-Tree相同，主要的区别是，B-Tree每个节点都会存储数据，而B+Tree只有叶子节点才会存储数据。

# 两者优势

## B+Tree优势

使用索引查找的时候计算机要先将数据结构读取到主存中，由于B+Tree的所有非叶子节点不存数据只存key，因此与B-Tree相比，使用B+Tree每次就能将更多的节点读取到主存中，也就能做更少的I/O操作。

## B-Tree优势

B-Tree由于每个节点都存数据，因此每个数据所在的层数是不同的，根节点只需查找一次，以此类推，根节点的子节点只需查找两次。 这样在一些场景下，开发者就可以通过将查找地频繁地数据放在靠近根节点的位置来优化查询。（类似于哈夫曼树）

# 使用B-Tree的硬件原因

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2412.png" alt="在这里插入图片描述"> 由于存储介质的特性，磁盘本身存取就比主存慢很多，再加上机械运动耗费，磁盘的存取速度往往是主存的几百分分之一，因此为了提高效率，要尽量减少磁盘I/O。为了达到这个目的，磁盘往往不是严格按需读取，而是**每次都会预读，即使只需要一个字节，磁盘也会从这个位置开始，顺序向后读取一定长度的数据放入内存**（即一个扇区）。 这样做的理论依据是计算机科学中著名的局部性原理：

>  
 当一个数据被用到时，其附近的数据也通常会马上被使用。 


因此，B-Tree的数据都存储在连续的硬盘空间，当去读取其中某个节点时，其连续存储的其他节点也会被读出来，这样就减少的I/O的次数。