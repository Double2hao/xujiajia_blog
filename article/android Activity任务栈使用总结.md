#android Activity任务栈使用总结
# 前言

android上对于任务栈的控制还是比较常见的，尤其是在小程序、小游戏的场景。 笔者近期也有在实际项目中碰到，因此就好好学习了一下，作此文以记录。 本文主要还是自己的一些使用总结，如有不对或者需要补充的地方，欢迎评论交流。

>  
 此文需要有对android启动模式的基础，还不了解的读者推荐看下笔者前面的文章：  


# 使用概念

android上要使用多任务栈，除了要了解启动模式之外，就是需要了解taskAffinity这个属性。

>  
 关于taskAffinity的官网地址： https://developer.android.com/guide/topics/manifest/activity-element#aff 


使用taskAffinity属性大概有以下几种场景：
1. 当前的taskAffinity与启动的taskAffinity相同。 这种情况下，无论什么情况都会启动在同一个任务栈中。这也是默认的情况。1. 当前的taskAffinity与启动的taskAffinity不同。 这种情况下，如果有FLAG_ACTIVITY_NEW_TASK的flag，那么会在新的任务栈中启动activity。如果没有设置这个flag，只要是启动模式中singleTask和singleInstance，那么也会在新的任务栈中启动activity。1. 启动的taskAffinity为空。 为空代表无论当前的activity是哪个任务栈，启动的activity都默认与其任务栈不相同。如果是带有FLAG_ACTIVITY_NEW_TASK的flag，或者启动模式为singleTask或singleInstance，那么肯定会启动一个新的任务栈。
# 启动模式对多个实例的支持
- 支持多个实例：standard和singleTop- 不支持多个实例：singleTask和singleInstance
默认一般都是支持多个实例的，主要讲下不支持多个实例的场景：
1. 逻辑非常独立，但是入口较多的页面。如小程序、小游戏。1. 需要使用到全局单例，全局不适合有多个的页面。如语音，视频。
# singleInstance最好要是单独的任务栈名称

假设主页面是A，B的启动模式是singleInstance并且任务栈名称与A相同。 那么按照如下操作，会有不合理的场景出现：
1. 首页启动A1. A启动B1. 回到首页再启动A
此时由于B是singleInstance，因此B不会在A的任务栈中。而又由于B是默认的任务栈名称，因此也没有自己单独的任务栈。 于是此时B虽然是存活的状态，但是却无法通过操作系统上的操作回到B。换句话说对用户来说B就是不可见的了。

>  
 为了避免这种情况的发生，除了给singleInstance设置单独的任务栈名称之外。也可以通过其他的一些方式，比如： 
 - 设置悬浮窗给用户保留进入B的入口。（这个需要权限）- 设置单独的桌面图标可以直接跳转到B。 


# 任务栈是否需要主要关闭

如果不主动关闭任务栈，那么用户还是可以再系统的任务栈预览页面重新进入的。 笔者还是列举了自己使用时碰到的两种常见：
1. 不需要主动关闭。 比如这个页面是小程序，我们期望用户第二次进入的时候要更快一点。那么在这种情况下，不仅不能关闭，而且最好关闭不要使用finish，而使用moveToBack。1. 主动关闭。 参数存在过期，或者不可用重复参数多次进入的页面。比如一些对战游戏会有房间的概念，在对战结束之后房间就没有了。这时候以过期的房间号进入页面是没有意义的，因此需要把任务栈主动关闭，避免用户主动操作触发这个错误的逻辑。
# Demo

笔者自己写了demo用于测试多任务栈下的各种场景，如有需要可以自行查看。

>  
 Demo github地址： 

