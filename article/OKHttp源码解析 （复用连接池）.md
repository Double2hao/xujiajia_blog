#OKHttp源码解析 （复用连接池）
>  
 原文地址：https://www.jianshu.com/p/6166d28983a2 


# 复用连接池

相关的主要三个类
1. RealConnection1. ConnectionPool1. StreamAllocation
>  
 对流的处理使用Okio，Okio是okhttp中用于处理io的模块，没接触过的读者可以参考这篇文章：https://www.jianshu.com/p/f5941bcf3a2d 


# RealConnection 要点
1. 代表着链接socket的链路，如果拥有了一个RealConnection就代表了我们已经跟服务器有了一条通信链路1. 实现了三次握手等操作。1. 一个链接对应多个流，链接和流的对应关系通过StreamAllocation来记录。
# ConnectionPool 要点

管理http和http/2的链接，以便减少网络请求延迟。同一个address将共享同一个connection。该类实现了复用连接的目标。
1. ConnectionPool内部以队列方式存储连接1. 连接池最多维持5个连接，且每个链接最多活5分钟1. 每次添加链接的时候回执行一次清理任务，清理空闲的链接。
# StreamAllocation 要点

StreamAllocation 协调了三者的关系：链接、流和回调。

>  
 这里谈一下对StreamAllocation的个人理解。 比如我要创建一个流，是需要当前的Connection的，甚至如果当前没有Connection，我是需要知道当前的ConnectionPool的。 这种逻辑的代码无论是写在RealConnection还是ConnectionPool还是Steam中都是不合理的，因此StreamAllocation中就处理了协调这些类的逻辑。 

1. 一个StreamAllocation中包括一个Steam和一个Connection。1. StreamAllocation中的方法包括：newStream（创建流），findHealthyConnection（找到可用的链接），streamFinished（关闭流），releaseIfNoNewStreams（释放链接）等等。