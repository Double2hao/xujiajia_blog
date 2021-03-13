#netty学习小结
# 前言

近期碰到网络相关的问题，准备使用netty，于是学习了一番，本文主要记录学习中的一些要点，至于诸多细节的学习，笔者推荐《Netty实战》和“简书闪电侠”的博客。

>  
 本文参考： 《netty实战》  


# netty简介

Netty是一个NIO客户端服务器框架，目前已经有苹果、Facebook等诸多大厂在使用。 主要特点：
1. API简单，文档丰富，社区活跃。1. 使用NIO，能用较少的线程实现高并发。
>  
 简单说下NIO的优势： NIO是非阻塞I/O模型，可以认为就是单个线程来处理所有的I/O事件。 我们常用的一般是一个Thread对应一个连接，处理I/O以及其他业务逻辑。在高并发的情况下，同时有多个线程同时使用网卡去读写数据的时候，其实是会阻塞的。 在这种情况下，如果使用单个线程来处理所有的I/O事件，那么就可以节省掉多个线程之间反复切换的性能消耗。 

1. 零拷贝
>  
 一般的发送数据是以下过程： 
 - 数据从磁盘读取到内核的read buffer- 数据从内核缓冲区拷贝到用户缓冲区- 数据从用户缓冲区拷贝到内核的socket buffer- 数据从内核的socket buffer拷贝到网卡接口（硬件）的缓冲区 
 Netty中只有两步： 
 - 调用transferTo,数据从文件由DMA引擎拷贝到内核read buffer- 接着DMA从内核read buffer将数据拷贝到网卡接口buffer 


# netty模型

netty常用的模型主要有两种，一种是单线程负责accept和I/O，另一种是两个线程分别负责accept和I/O，下图来源于Doug Lee大神的 Reactor 介绍：

### 单线程负责accept和I/O
1. Reactor线程负责接收连接和 I/O（一般也包括协议的编解码，比如HTTP协议，HTTPS协议）1. 如果连接在read之后需要有其他业务处理，比如计算之类可能耗时的操作，可以直接自定义一个线程池去做。（一般情况下编解码操作不像图中显示的一样会在自定义的线程池中做，而是通过ChannelPipeline直接在I/O的时候也在Reactor线程中做掉了）
<img src="https://img-blog.csdnimg.cn/20190615143337781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

### 两个线程分别负责accept和I/O
1. mainReactor线程负责接收连接1. subReactor线程一般会设置多个，负责I/O（一般也包括协议的编解码，比如HTTP协议，HTTPS协议）1. 如果连接在read之后需要有其他业务处理，比如计算之类可能耗时的操作，可以直接自定义一个线程池去做。（一般情况下编解码操作不像图中显示的一样会在自定义的线程池中做，而是通过ChannelPipeline直接在I/O的时候也在subReactor线程中做掉了） <img src="https://img-blog.csdnimg.cn/20190615143401534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">
# netty关键类

### Channel

顾名思义，就是通道的意思，可以认为它负责与连接相关的参数和操作，它的作用主要有两点：
1. 获取与连接相关的一些参数，比如连接的状态，连接的配置参数等。1. 可以通过它实现与连接相关的操作，比如建立连接，读写操作等。
>  
 一些常用的 Channel 类型： 
 - NioSocketChannel，异步的客户端 TCP Socket 连接。- NioServerSocketChannel，异步的服务器端 TCP Socket 连接。- NioDatagramChannel，异步的UDP 连接。 


### EventLoop

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/372.png" alt="在这里插入图片描述"> EventLoop是netty实现nio的关键，EventLoop在netty中一般就用来处理accpt事件和I/O事件。 一个EventLoop对应一个线程，这个线程通过循环来查看是否有需要处理的事件，如果有，那么就拿出来处理。 EventLoop中Selector和taskQueue分别负责两种事件： 1、Selector是netty内部使用的，开发者一般不会用到。它只负责与“Channel”相关的事件，比如负责accept，read和write等。 2、taskQueue有两种情况下会使用到，第一种是异步抛一个定时任务给EventLoop处理，第二种是多线程协作的时候会用到，比如其他线程抛一个I/O事件给负责I/O的EventLoop，就是添加了一个task到该EventLoop的taskQueue中。

>  
 write事件在源码中的逻辑是这样的： 
 - 先判断当前是不是I/O的EventLoop，如果是，那么就直接执行write逻辑。- 如果当前不是，那么就把write事件封装成一个task投递到负责I/O的EventLoop的taskQueue中。 


### EventLoopGroup

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/373.png" alt="在这里插入图片描述"> 顾名思义EventLoopGroup就是EventLoop的Group，其中可以包括一个或者多个EventLoop。 在netty中，用于accept的EventLoopGroup一般只会有一个EventLoop，而用于I/O的EventLoopGroup一般会有多个EventLoop。 由于EventLoop和线程是一一对应的关系，所以I/O的EventLoopGroup有多个EventLoop也意味着有多个线程会处理I/O事件。

### SelectedSelectionKeySet

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/374.png" alt="在这里插入图片描述"> 执行Selector.select()，如果有事件，那么就会将事件作为SelectionKey加入到SelectedSelectionKeySet中。 SelectionKey会带有一个attachment，一般来说，这个attachment就是Channel。 然后就直接通过使用Channel来处理I/O事件。

### ChannelHandler和ChannelPipeline

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/375.png" alt="在这里插入图片描述"> ChannelPipeline就是ChannelHandler的一个List。 在通过Channel去执行write和read操作的时候，会通过ChannelPipeline来执行一些与I/O相关的操作，比如HTTP编码解码，SSL的加密解密等。

值得注意的是，ChannelPipeline在出站和入站的时候，ChannelHandler的执行顺序是不同的。 在出站的时候是从head到tail，在入站的时候是从tail到head。 另外，入站的时候只执行ChannelInboundHandler，出站的时候只执行ChannelOutboundHandler。 <img src="https://img-blog.csdnimg.cn/20190615165106605.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

>  
 举个例子： 比如现在ChannelPipeline中ChannelHandler如下： 
 - decodeChannelHandler（ChannelInboundHandler）- encodeChannelHandler（ChannelOutboundHandler）- computeReadChannelHandler（ChannelInboundHandler）- computeWriteChannelHandler（ChannelOutboundHandler） 在入站的时候，执行的顺序就是- decodeChannelHandler（ChannelInboundHandler）- computeReadChannelHandler（ChannelInboundHandler） 在出站的时候，执行的顺序就是- computeWriteChannelHandler（ChannelOutboundHandler）- encodeChannelHandler（ChannelOutboundHandler） 


# 关键类模型

笔者根据关键类，也画了下netty模型，可以供参考。 <img src="https://img-blog.csdnimg.cn/20190615174239179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# EventLoop关键逻辑

EventLoop.run()源码如下：

```
protected void run() {
        for (;;) {
            oldWakenUp = wakenUp.getAndSet(false);
            try {
                if (hasTasks()) {
                    selectNow();
                } else {
                    select();
                    if (wakenUp.get()) {
                        selector.wakeup();
                    }
                }

                cancelledKeys = 0;

                final long ioStartTime = System.nanoTime();
                needsToSelectAgain = false;
                if (selectedKeys != null) {
                    processSelectedKeysOptimized(selectedKeys.flip());
                } else {
                    processSelectedKeysPlain(selector.selectedKeys());
                }
                final long ioTime = System.nanoTime() - ioStartTime;

                final int ioRatio = this.ioRatio;
                runAllTasks(ioTime * (100 - ioRatio) / ioRatio);

                if (isShuttingDown()) {
                    closeAll();
                    if (confirmShutdown()) {
                        break;
                    }
                }
            } catch (Throwable t) {
                logger.warn("Unexpected exception in the selector loop.", t);

                // Prevent possible consecutive immediate failures that lead to
                // excessive CPU consumption.
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // Ignore.
                }
            }
        }
    }

```

源码不过多分析，直接说结果，有兴趣的读者可以自己看下。 EventLoop.run()中最终走向的主要是三个方法：
1. selector.select() 负责轮询出I/O事件，将事件作为SelectionKey加入到SelectedSelectionKeySet中。1. processSelectedKey() 从SelectedSelectionKeySet中取出SelectionKey，根据其Channel来进行read，write等操作。ChannelPipeline中的ChannelHandler就是在这个过程中被执行到的。1. runAllTask() 运行taskQueue中的事件，事件一般有两种，第一种是异步定时事件，第二种是是其他线程发来的I/O事件。