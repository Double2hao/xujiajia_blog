#解决android studio adb not responding 的问题
Android studio与Eclipse相较之下还是人性化很多的。Eclipse中出现了ADB的占用问题需要使用kill-server和start-server来重启ADB，而Android studio中则是省去了开发者自己去运行cmd的时间了。如下图：

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039406040.png">

 

 

然而这种Android studio的人性化设计也不是万能的，总会有一些意外情况，我近期便遇到了一例，接下来便说一下解决办法。

 

首先还是需要打开CMD，adb运行占用的端口号是5037所以便**输入**netstat -ano | findstr "5037"，如下图：

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039407301.png">

 

如图可以看到，是有PID为7336的进程占用了ADB。接下来很简单，打开任务管理器把该进程干掉就可以了。

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039408822.png">

 

本人博客、Android均为新手，闻过则喜，望各位前辈不吝批评指点。