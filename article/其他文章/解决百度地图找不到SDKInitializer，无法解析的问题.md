#解决百度地图找不到SDKInitializer，无法解析的问题
笔者根据网友的博客，在使用百度地图的时候导入jar与so文件，如图：

 

 

 

**然后在jar已经导入的情况，出现了SDKInitializer无法解析的问题，也就是没有SDKInitializer这个类。**

笔者在百度了近两个小时之后，发现很少有人碰到这个问题，还是没有解决，然后在左思右想之后，今早打开AS重新弄了一下，便解决了。

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910972140.png ">

如何？

在网上，大多的教程都是教我们只导入一个jar和一个so（顶多增加一个定位服务），但是新的百度的API有众多的jar与so文件，如图：

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910973011.png ">

 

笔者之前也就是和老教程中一样，只选择了baidumapapi_map_v3_6_1这个jar导入，so文件中也只选择了这个，然后就会出现**找不到SDKInitializer**的问题。

只需要把所有的jar与so文件全部导入，问题马上就解决了。（另外，大家可以看一下，baidumapapi_map_v3_6_1这个文件的内存是最大的，其他的都比较小，全部导入也不会对项目大小有太大影响。当然如果对API有深入理解的，用多少拿多少还是最好的。）

 

最后附上可以使用了的效果图：

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910973912.png ">

 

 

**最后笔者也是啰嗦一句，最好从下下来的百度地图的demo里面的copy jar与so文件，笔者在使用的时候，直接使用下载的jar和so有问题，具体原因未知。**