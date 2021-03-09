#bindService 源码解析（为什么是异步）
# 概述

>  
 Andromeda 源码解析 （同步获取服务）： 


前一篇文章分析了Andromeda的源码，其中提到，bindService获取IBinder对象的操作是异步的。 那么为什么会是异步的呢，其中做了哪些操作呢？

>  
 在此推荐下看Android源码网站： （源码较多的文件建议下载后再本地查看，网页上看会比较卡） 


# onServiceConnected的调用栈

首先直接在onServiceConnected中输出一下调用栈。

```
     at com.example.bindservicetest.MainActivity$1.onServiceConnected(MainActivity.java:29)
     at android.app.LoadedApk$ServiceDispatcher.doConnected(LoadedApk.java:1956)
     at android.app.LoadedApk$ServiceDispatcher$RunConnection.run(LoadedApk.java:1988)
     at android.os.Handler.handleCallback(Handler.java:883)
     at android.os.Handler.dispatchMessage(Handler.java:100)
     at android.os.Looper.loop(Looper.java:224)
     at android.app.ActivityThread.main(ActivityThread.java:7520)
     at java.lang.reflect.Method.invoke(Native Method)
     at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:539)
     at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:950)

```

可以看到，最终是调用到了RunConnection这个Runnable。 是通过handle调用的，因此执行是异步的。 那么在bindService中哪里会执行到呢？

# onServiceConnected在bindService中的调用

由于源码较多，笔者整理了一下，几种进入onServiceConnected的调用顺序。

### Service已经启动的时候 调用流程
1. ContextWrapper.bindService()1. ContextImpl.bindService()1. ContextImpl. bindServiceCommon()1. ActivityManagerService.bindService() （这里是IPC调用，是异步的）1. ActiveServices.bindServiceLocked()1. LoadedApk.ServiceDispatcher.InnerConnection.connected() （这里是IPC调用，是异步的）1. LoadedApk.ServiceDispatcher.connected()1. ActivityThread.post() (这里使用到了handle，后面异步)1. LoadedApk.ServiceDispatcher. RunConnection.run()1. LoadedApk.ServiceDispatcher.doConnected()1. ServiceConnection.onServiceConnected()
### Service是首次启动 调用流程
1. ContextWrapper.bindService()1. ContextImpl.bindService()1. ContextImpl. bindServiceCommon()1. ActivityManagerService.bindService() （这里是IPC调用，是异步的）1. ActiveServices.bindServiceLocked()1. ActiveServices.requestServiceBindingLocked()1. ActivityThread.ApplicationThread.scheduleBindService()1. ActivityThread.sendMessage() (这里使用到了handle，后面异步)1. ActivityThread.handleBindService()1. ActivityManagerService.publishService() （这里是IPC调用，是异步的）1. ActiveService.publishServiceLocked()1. LoadedApk.ServiceDispatcher.InnerConnection.connected() （这里是IPC调用，是异步的）1. LoadedApk.ServiceDispatcher.connected()1. ActivityThread.post() (这里使用到了handle，后面异步)1. LoadedApk.ServiceDispatcher. RunConnection.run()1. LoadedApk.ServiceDispatcher.doConnected()1. ServiceConnection.onServiceConnected()
### 具体分析

根据上面的调用顺序，其实大致可以定位异步的原因，即：

**InnerConnection的IPC调用，导致回调后的逻辑在binder线程中，因此需要ActivityThread将事件post到主线程执行。**

>  
 为什么InnerConnection会用到IPC？ 由于createService和bindService是通过IPC来让ActivityManagerService实现的，所以在连接建立之后需要再通过IPC来告诉clinet端结果。 

