#由浅入深 学习 Android Binder（十）- 总结
# 概述

耗时两三个月，Android Binder系列终于整理完了。

### 个人一点感想

笔者在大学学习Android时也一直对Binder非常有兴趣，奈何多次探索，多次无功而返。 现在想想，不仅是缺乏Binder实践的经验，更多的也是故步自封，没有考虑过直接剖析native层源码来理解binder。（那时候总想着java层都还没搞明白，直接看native层没有意义）

后来随着工作经验的积累，对Binder和多进程也有一定的实践经验，再次回顾时，发现很多细节也是可以理解了，于是就开始思考如何去整理Binder这一些系列的知识。

我个人非常明白，不能”直接开始解析某个类“地来写，这样的后果就是”懂得人不用看，需要的人看不懂“。 最终考虑到大部分的开发者其实更多专注于上层的开发，所以打算从平时大家使用最多的一些东西由浅入深地来讲，比如AIDL，比如binderService。

期望自己的整理能帮助到需要的人。

# 系列文章

>  
            


# Binder应用 文章

Binder应用类的文章是在此系列之前笔者记录的一些个人使用经验，有兴趣的读者也可以看下：  

# 继续探索

笔者写此系列，只是根据自己经验整理一些自己觉得常见的点。 自己也深知，与Binder的整体知识相比不过是冰山一角。

Binder相关的知识点，比如AMS，PMS，WMS等，笔者后面也会继续学习一部分，只是什么时候整理出来就是不可预期了。

另外，此系列没有解析Binder driver，对其有兴趣的非常推荐看下此篇文章： 