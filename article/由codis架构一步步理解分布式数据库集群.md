#由codis架构一步步理解分布式数据库集群
# 前言

近期在搭建codis，其中涉及的知识点较多，可以说每个控件都有其作用。而很多知识点在分布式数据库集群的方向上是共通的。 于是，在大概梳理完知识点后，做此文记载该块内容，也供后面入坑的新手参考。

>  
 codis github链接： 


# 主从服务器和sentinel

我们要搭建一个数据库服务器，最简单的就是直接一台服务器部署数据库的功能，如下： <img src="https://img-blog.csdnimg.cn/20190330160659749.png" alt="在这里插入图片描述"> 后来发现这样部署会有一个问题，如果数据库服务器意外宕机，数据就会丢失，并且服务也会中断。 而为了避免数据丢失和服务中断，就引入了“主从服务器”和sentinel，如下图： <img src="https://img-blog.csdnimg.cn/20190330161924403.png" alt="在这里插入图片描述"> sentinel一直监控master服务器，查看是否宕机，如果宕机了，就从slave服务器中选出一台作为新的master服务器，这样就能保证服务器的功能一直不会断。（这里主从切换用到的知识是“VIP漂移”） 当然sentinel也是可能会宕机的，所以要部署多台sentinel，多台sentinel之间也可以互相监控是否宕机。

# codis-group

上面的整个系统在coids中称之为 codis-group。因为用了“VIP漂移”，所以group对外的IP是固定的，用户就直接可以通过这个虚拟IP来访问到实际可用的服务器。 架构就如下图： <img src="https://img-blog.csdnimg.cn/20190403195127964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# codis-proxy

在这样的模式下，只有单个服务器在抗所有的访问，随着数据量和并发量的增多，需要横向增添服务器。于是就又引入了codis-proxy的概念。 coids-proxy会同时连接多个coids-group，codis-proxy会把数据库操作均衡地分发给每一个group，这样就可以通过横向增加服务器来缓解服务器并发与数据量上的压力。 架构如下图： <img src="https://img-blog.csdnimg.cn/20190330163205227.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# 哈希分片

根据上面的图片，有的读者会思考，那么代理服务器是如何能将请求平均分布多个group上的呢？ 这个概念称为数据分片，在codis中是使用的slot，翻译过来可以称为虚拟桶。 由于笔者之前写过哈希分片的文章，在此文就不徒增篇幅了，有兴趣的读者可以看笔者哈希分片的文章。

>  
  


# zookeeper

代理服务器也存在两个问题： 1、本身可能宕机 2、每条消息进来需要判断分配到哪个group，这带来了部分性能消耗，在高并发的情况下，单机可能扛不住。 为了解决这两个问题，所以又引入了zookeeper系统，如下图： <img src="https://img-blog.csdnimg.cn/20190330164859944.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> zookeeper系统负责管理多个代理服务器，用户直接从zookeeper系统去拿可用的代理服务器地址，如果主机宕机了从机顶上，这样就解决了代理服务器宕机的问题。 zookeeper支持读写分离，对于大多数服务来讲，“读”一定是比“写”多的，也为了保证安全，zookeeper只允许主机进行写操作，它会将读操作分配到多台代理服务器的从机上，这样就解决了哈希分片给代理服务器带来的性能消耗问题。

当然zookeeper系统中的服务器也是可能宕机的，因此也需要设置多台。

# codis-dashboard

到这里其实整个集群已经比较完备了。 为了运营，也必须添加管理代理服务器，管理group和sentinel的入口，这就是codis-dashboard，如下图： <img src="https://img-blog.csdnimg.cn/20190330165554826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> codis-dashboard只是管理集群的入口，因此不需要考虑宕机的问题，它宕机的也完全不会影响集群的正常运行啊。 而且codis-dashboard也并不需要一直运行着，需要配置集群内容的时候再把它运行起来也完全可以。 但是可能会有同时多个人在进行运维，而不仅仅只有一台机器。因此，需要将整个集群的信息存储到云端（在codis中称为外存储），这样每个人配置的操作都会云端同步。

# codis-fe

codis-fe的概念就很简单了，就是codis-dashboard的可视化前端，通过codis-fe直接可以通过在网站上的操作来更改集群配置。 不过因为codis-fe只是个前端可视化界面，所以codis-fe在codis集群中不是必须的。

# 总结

这时候再来看下codis 官网文档上的架构图就能很清晰的理解了，如下： <img src="https://img-blog.csdnimg.cn/20190330170442638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">
