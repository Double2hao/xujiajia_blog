#Android ActivityManagerService (AMS)总结
# 转载前言

AMS在Android中算是比较重要的一个知识点，不管是binder的源码还是Serveice的源码，都是涉及到AMS的，因此理解“AMS是什么”对一个Android开发者来说还是很有必要的。

>  
 笔者之前也有分析过相关的源码： 


已经有博主总结的比较好了，因此就不重开篇幅了。

>  
 原文地址：https://github.com/xiangjiana/Android-MS/blob/master/android/ams.md 


# 概述

ActivityManagerService是Framework层的核心服务之一,ActivityManagerService是Binder的子类,它的功能主要以下三点:
- 四大组件的统一调度- 进程管理- 内存管理
# 四大组件的统一调度

ActivityManagerService最主要的功能就是统一的管理者activity,service,broadcast,provider的创建,运行,关闭.我们在应用程序中启动acitivity,关闭acitiviy等操作最终都是要通过ams来统一管理的.这个过程非常的复杂,不是一下子可以讲的清楚的,我这里推荐老罗的博客来讲解四大组件的启动过程:
- Android应用程序内部启动Activity过程（startActivity）的源代码分析- Android系统在新进程中启动自定义服务过程（startService）的原理分析- Android应用程序注册广播接收器（registerReceiver）的过程分析- Android应用程序发送广播（sendBroadcast）的过程分析- Android应用程序组件Content Provider简要介绍和学习计划
# AMS中的进程管理

前面曾反复提到，Android平台中很少能接触到进程的概念，取而代之的是有明确定义的四大组件。但是作为运行在Linux用户空间内的一个系统或框架，Android不仅不能脱离进程，反而要大力利用Linux OS提供的进程管理机制和手段，更好地为自己服务。作为Android平台中组件运行管理的核心服务，ActivityManagerService当仁不让地接手了这方面的工作。目前，AMS对进程的管理仅涉及两个方面：
- 调节进程的调度优先级和调度策略。- 调节进程的OOM值。
### App的Crash处理总结

应用进程进行Crash处理的流程。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/3210.png" alt="在这里插入图片描述">

# 内存管理

我们知道当一个进程中的acitiviy全部都关闭以后,这个空进程并不会立即就被杀死.而是要等到系统内存不够时才会杀死.但是实际上ActivityManagerService并不能够管理内存,android的内存管理是Linux内核中的内存管理模块和OOM进程一起管理的.Android进程在运行的时候,会通过Ams把每一个应用程序的oom_adj值告诉OOM进程,这个值的范围在-16-15,值越低说明越重要,越不会被杀死.当发生内存低的时候,Linux内核内存管理模块会通知OOm进程根据AMs提供的优先级强制退出值较高的进程.因此Ams在内存管理中只是扮演着一个提供进程oom_adj值的功能.真正的内存管理还是要调用OOM进程来完成.下面通过调用Activity的finish()方法来看看内存释放的情况.

当我们手动调用finish()方法或者按back键时都是会关闭activity的,在调用finish的时候只是会先调用ams的finishActivityLocked方法将当前要关闭的acitiviy的finish状态设置为true,然后就会先去启动新的acitiviy,当新的acitiviy启动完成以后就会通过消息机制通知Ams,Ams在调用activityIdleInternalLocked方法来关闭之前的acitiviy.

# startService流程

**总结** 在整个startService过程，从进程角度看服务启动过程
- **Process A进程**：是指调用startService命令所在的进程，也就是启动服务的发起端进程，比如点击桌面App图标，此处Process A便是Launcher所在进程。- **system_server进程**：系统进程，是java framework框架的核心载体，里面运行了大量的系统服务，比如这里提供ApplicationThreadProxy（简称ATP），ActivityManagerService（简称AMS），这个两个服务都运行在system_server进程的不同线程中，由于ATP和AMS都是基于IBinder接口，都是binder线程，binder线程的创建与销毁都是由binder驱动来决定的，每个进程binder线程个数的上限为16。- **Zygote进程**：是由init进程孵化而来的，用于创建Java层进程的母体，所有的Java层进程都是由Zygote进程孵化而来；- **Remote Service进程**：远程服务所在进程，是由Zygote进程孵化而来的用于运行Remote服务的进程。主线程主要负责Activity/Service等组件的生命周期以及UI相关操作都运行在这个线程； 另外，每个App进程中至少会有两个binder线程 ApplicationThread(简称AT)和ActivityManagerProxy（简称AMP），当然还有其他线程，这里不是重点就不提了。
<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/3211.png" alt="在这里插入图片描述">

图中涉及3种IPC通信方式：Binder、Socket以及Handler，在图中分别用3种不同的颜色来代表这3种通信方式。一般来说，同一进程内的线程间通信采用的是 Handler消息队列机制，不同进程间的通信采用的是binder机制，另外与Zygote进程通信采用的Socket。

**启动流程**：
1. Process A进程采用Binder IPC向system_server进程发起startService请求；1. system_server进程接收到请求后，向zygote进程发送创建进程的请求；1. zygote进程fork出新的子进程Remote Service进程；1. Remote Service进程，通过Binder IPC向sytem_server进程发起attachApplication请求；1. system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向remote Service进程发送scheduleCreateService请求；1. Remote Service进程的binder线程在收到请求后，通过handler向主线程发送CREATE_SERVICE消息；1. 主线程在收到Message后，通过发射机制创建目标Service，并回调Service.onCreate()方法。 到此，服务便正式启动完成。当创建的是本地服务或者服务所属进程已创建时，则无需经过上述步骤2、3，直接创建服务即可。
<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/3212.png" alt="在这里插入图片描述">