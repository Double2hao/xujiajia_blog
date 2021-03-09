#SkipList详解
>  
 本文参考：《大数据日知录》 


# 概念

SkipList是一种用来代替平衡树的数据结构。 虽然在最坏的情况下SkipList的效率要低于平衡树，但是大多数情况下效率仍然非常高，其插入、删除、查找的时间复杂度都是O(log(N))。 除了高效外，其实现和维护非常简单也是一大优势。

SkipList的使用还是比较广泛的，比如在LevelDB中的MemTable就是使用SkipList实现的，Redis的Sorted Set也是使用SkipList实现的。

# 核心思路

在正常的有序列表中，我们要查找到一个值的时间复杂度是O(N)。 <img src="https://img-blog.csdnimg.cn/20190519171942931.png" alt="在这里插入图片描述"> 如果链表中一半节点可以多保留一个指针指向后继节点的后继节点，那么查找的时间复杂度就可以变成O(N/2)。 <img src="https://img-blog.csdnimg.cn/20190519172203530.png" alt="在这里插入图片描述"> 以此类推，如果保留后面三个节点的指针，查找的时间复杂度就是O(N/3)。

至此，SkipList的核心思路就出来的，就是通过保留后面多个节点的指针来提高查找的效率，下图是一个典型的SkipList结构。 <img src="https://img-blog.csdnimg.cn/20190519172415415.png" alt="在这里插入图片描述">

# SkipList节点的生成

SkipList中有一个MaxLevel的值，每个节点会随机生成一个1~MaxLevel的值，这个值决定了这个节点有多少个指向后继节点的指针。 如果这个随机值是4，那么这个节点就会有指向后面4个节点的指针。

>  
 MaxLevel是可以主动设置的，在Redis中这个MaxLevel默认是64。 MaxLevel设置太大并没有什么意义，具体的还是要看length有多大。 


# SkipList的查找

<img src="https://img-blog.csdnimg.cn/20190519172415415.png" alt="在这里插入图片描述"> 还是以此图为例，比如现在要找12这个节点，那么过程如下：
1. 首先到3的节点，12&gt;3，所以进入6的节点。1. 在6的节点，26&gt;6，并且12&lt;25，12&gt;9，所以进入9的节点。1. 在9的节点，12&lt;17，12=12，于是就找到了12的节点。
# SkipList的插入

<img src="https://img-blog.csdnimg.cn/20190519173738184.png" alt="在这里插入图片描述"><img src="https://img-blog.csdnimg.cn/20190519173851271.png" alt="在这里插入图片描述"> 以上图为例，现在要插入17这个节点。 其过程和查找类似，唯一的问题是，前面的节点的指针是如何保留下来的？

我们可以看到插入结束后，9的level=1的指针指向了17，12的level=0的指针指向了17。 这就意味着，在插入的时候我们就需要保留9的level=1的指针和12的level=0的指针。

在SkipList中是这样做的，有一个update数组，这个数组的大小为maxLevel。 还是以上图为例，在17插入之前，update这个数组中就已经存储了当然每个level的指针。
- update[0]：12 level=0- update[1]：9 level=1- update[2]：6 level=2- update[3]：6 level=3
# SkipList的删除

删除的逻辑和插入类似：
1. 查找到相应的节点1. 通过update数组来实现该节点的逻辑删除1. 回收该节点资源