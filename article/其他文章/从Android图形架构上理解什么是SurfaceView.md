#从Android图形架构上理解什么是SurfaceView
# 概念

SurfaceView的内容可以在单独的线程和单独的进程中渲染。

# 使用场景

拿斗鱼APP举个例子，如下图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/3090.png" alt="在这里插入图片描述"> 视频肯定是用的SurfaceView，这样就可以单独线程或者进程渲染。

如果不用单独渲染会有什么问题？ 可以看到下面会有一个聊天框，在用户特别多的时候，下面的聊天框肯定渲染非常频繁，而视频也需要及时渲染，如果两者在同一个线程，那么频繁渲染的聊天框很可能会影响到视频的渲染。 这样用户只要一多，视频就会显示卡顿，显然这样是不符合预期的。

# 图形架构上什么是SurfaceView

下面是Android图形架构，如果不理解的读者可以看笔者上一篇文章：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/3091.png" alt="在这里插入图片描述">

Android图形架构中，有多个Surface，每个Surface之间的渲染不会互相影响，常见的Surface有主屏幕的Surface，状态栏的Surface等等。 而使用SurfaceView则会有一个单独的Surface，因此SurfaceView的渲染可以和其他Surface不会互相影响，也就是和主屏幕以及状态栏等的渲染不会互相影响。

# 注意点：SurfaceView的生命周期

由于SurfaceView有自己单独的Surface，和App主线的View不会在一起渲染，因此SurfaceView的生命周期也是独立于其他View之外的。 打个比方： 在Activity中有一个SurfaceView，在Activity onPause()，onDestroy()的时候，SurfaceView不会和其他View一样与Activity的生命周期同步。就拿在Activity onPause()的时候来说，SurfaceView不会管Activity的onPause()，继续执行自己的渲染逻辑。

因此在使用SurfaceView的时候一定要注意同步SurfaceView和主线程的View的生命周期。