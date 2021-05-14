#Android 图形架构简介
>  
 本文参考： 


# 流程

一、无论开发者使用什么渲染 API，一切内容都会渲染到Surface。（常见API：canvas，openGL） 二、Surface把图像流缓存到buffer queue 三、SurfaceFlinger 从多个buffer queue中去获取图像流执行合并操作 四、 Hardware Composer 去获取SurfaceFlinger缓存的内容实现上屏操作 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1460.png" alt="在这里插入图片描述">

# 一、OpenGL渲染流程

这个流程网上讲解很多，此处就不多加篇幅了： 1、读取顶点数据 2、执行顶点着色器 3、组装图元 4、光栅化处理 5、片段着色器（这里涉及常见的二次线性插值，抗锯齿等）

# 二、为什么Surface要有个缓冲区

试想一下，Surface同时会发生有读写操作，如果没有缓冲区，那么就只能加锁。 举个例子，当SufaceFlinger去获取Surface的图像流时，因为锁的原因阻塞住了，那么整个合并的操作就会阻塞住，用户所看到的就是整个屏幕卡住了，这显然是不合理的。 比如，此刻有两个Surface，一个是状态栏的Surface，一个是主屏幕的Surface，当主屏幕因为逻辑原因卡住的时候，那么不应该会影响到状态栏。

# 三、为什么要有SurfaceFlinger的合并操作

一句话其实就是“不能没有统一管理”。 图像流上屏时，对于硬件来讲，它并不知道哪一部分属于哪个View或者属于哪个进程，它只会每一帧将整个屏幕中的所有像素刷新。 试想如果每个进程或者每个View都不需合并直接去操作进程，那么很可能一个View正在上屏，而另一个View就发来了上屏请求，屏幕很可能上一个View还没更新完，就要开始更新下一个，那么就会出现帧撕裂的情况，如下图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1461.png" alt="在这里插入图片描述">

# 四、上屏显示的过程

这个过程下面的文章讲的很不错，此处直接附上文章地址： 

下图摘自该文章： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1462.png" alt="在这里插入图片描述">