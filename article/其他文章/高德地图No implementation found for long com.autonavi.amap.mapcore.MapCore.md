#高德地图No implementation found for long com.autonavi.amap.mapcore.MapCore
此篇博客最后更新时间写自2016.5.18。当下高德地图jar版本为3.3.1。

 

使用高德地图碰到此问题，纠结许久（接近4个多小时）。

记录在此，希望遇到相同问题的读者可以有所借鉴。

 

错误截图：

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1680.png">

 

导致问题的原因主要有两种：

**1、so文件操作不当问题。可能是so文件与jar不匹配，也可能是so文件未导入。**

解决办法：

下载与jar相匹配的jar。sdk下载网址：

 

 

**2、由于在X86手机上运行，而此时高德地图官网并未发布X86的so文件，导致应用崩溃。**

（注：android4.4之后的大部分机型都是X86的）

 

解决办法：

 

**只保留armeabi****文件夹**，其他的统统删掉，因为大多数x86平台的手机都会兼容armeabi的版本。

但是会发现就算这样做了在**模拟器上面依旧装不上**，那是**因为模拟器没有兼容**，但是他可以替换平台。如果是genymotion的话，需要安装一个转换为arm的插件。

 

 

 

针对第二个问题，官网的配置工程也是有提到的，如下图：

（网址：）

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1681.png">

 

 

参考网址：

 