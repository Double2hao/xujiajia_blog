#由浅入深 学习 Android Binder（七）- IServiceManager与ServiceManagerNative（java层）
>  
 Android Binder系列文章：           


# 概述

ServiceManager于android binder来说是非常重要的一部分。 ServiceManager在java层与native层都有各自的逻辑，本文主要讲下java层ServiceManager的逻辑。

>  
 如果了解过binder的读者，可能也多少会听说过binder与serviceManager的关系 


为了方便读者有个整体的认知，附上整体的流程图： <img src="https://img-blog.csdnimg.cn/20210103085615242.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# 何处使用了ServiceManager

前文中，在client与ActivityManagerService通信的时候，会通过ServiceManager去获取IActivityManager对象，代码在ActivityManager中，如下：

```
    public static IActivityManager getService() {<!-- -->
        return IActivityManagerSingleton.get();
    }

    private static final Singleton&lt;IActivityManager&gt; IActivityManagerSingleton =
            new Singleton&lt;IActivityManager&gt;() {<!-- -->
                @Override
                protected IActivityManager create() {<!-- -->
                    final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                    final IActivityManager am = IActivityManager.Stub.asInterface(b);
                    return am;
                }
            };


```

那么ServiceManager是如何获取到IActivityManager的呢？ 我们直接看下ServiceManager.getService()的源码。

# ServiceManager.getService()

```
    public static IBinder getService(String name) {<!-- -->
        try {<!-- -->
            IBinder service = sCache.get(name);
            if (service != null) {<!-- -->
                return service;
            } else {<!-- -->
                return Binder.allowBlocking(rawGetService(name));
            }
        } catch (RemoteException e) {<!-- -->
            Log.e(TAG, "error in getService", e);
        }
        return null;
    }

```

这里的逻辑是，先去缓存中获取，如果缓存中没有就返回rawGetService()。

缓存逻辑与我们所要探究的内容无关，此处就先不看了。 直接进入到rawGetService（）.

# ServiceManager.rawGetService()

```
    private static IBinder rawGetService(String name) throws RemoteException {<!-- -->
——————————————————省略
        final IBinder binder = getIServiceManager().getService(name);
        
——————————————————省略
        return binder;
}

```

直接进入到ServiceManager.rawGetService()。 这里的逻辑是，先通过getIServiceManager()获取到IServiceManager对象，然后再通过这个IServiceManager对象获取到一个IBinder。

注意，此处有两个IPC
1. 获取到IServiceManager1. 通过IServiceManager获取到该name的IPC
我们先看下getIServiceManager()的逻辑。IServiceManager.getService()的逻辑后面也会再提到

# ServiceManager.getIServiceManager()

```
    private static IServiceManager getIServiceManager() {<!-- -->
        if (sServiceManager != null) {<!-- -->
            return sServiceManager;
        }

        // Find the service manager
        sServiceManager = ServiceManagerNative
                .asInterface(Binder.allowBlocking(BinderInternal.getContextObject()));
        return sServiceManager;
    }

```

到这，可以看到IServiceManager其实就是ServiceManagerNative.asInterface()的返回值。

BinderInternal.getContextObject()中返回的是一个BinderProxy对象。本文由于篇幅限制，就不重复讨论，有兴趣的可以看笔者前文的详细探索： 

最终我们需要看的其实就是ServiceManagerNative.asInterface()这个方法。

# ServiceManagerNative.asInterface()

这里client与ServiceManagerNative不在一个进程，因此最终返回的是ServiceManagerProxy对象。

>  
 对于AIDL的逻辑不是很了解读者可以看下笔者前文：  


```
    static public IServiceManager asInterface(IBinder obj)
    {<!-- -->
        if (obj == null) {<!-- -->
            return null;
        }
        IServiceManager in =
            (IServiceManager)obj.queryLocalInterface(descriptor);
        if (in != null) {<!-- -->
            return in;
        }

        return new ServiceManagerProxy(obj);
    }

```

# ServiceManagerProxy逻辑

前面ServiceManager中，通过获取IServiceManager.getService()来获取到IActivityManager，我们此处就看下ServiceManagerProxy的getService逻辑。

```
    private static IBinder rawGetService(String name) throws RemoteException {<!-- -->
——————————————————省略
        final IBinder binder = getIServiceManager().getService(name);
        
——————————————————省略
        return binder;
}

```

getService主要逻辑如下：
1. 将数据都通过Parcel对象写入1. 调用mRemote.transact()来进行IPC操作，传过去的code是GET_SERVICE_TRANSACTION。1. 从reply中获取Ibinder对象。（Context.ACTIVITY_SERVICE对应的Ibinder就是IActivityManager）
```
class ServiceManagerProxy implements IServiceManager {<!-- -->
    public ServiceManagerProxy(IBinder remote) {<!-- -->
        mRemote = remote;
    }

    public IBinder asBinder() {<!-- -->
        return mRemote;
    }

    public IBinder getService(String name) throws RemoteException {<!-- -->
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IServiceManager.descriptor);
        data.writeString(name);
        mRemote.transact(GET_SERVICE_TRANSACTION, data, reply, 0);
        IBinder binder = reply.readStrongBinder();
        reply.recycle();
        data.recycle();
        return binder;
    }
    
————————————————————————————省略
}

```

# ServiceManagerNative.onTransact不会被调用

在ServiceManagerNative中，它的onTransact方法不会被调用。

对aidl有一定了解的读者，会知道，在aidl定义的binder对象中，Stub.onTransact和Stub.Proxy中的IBinder.transact 会成对存在。即，当client进程调用了transact之后，server进程会调用到onTransact方法。

>  
 前文aidl文章地址： 


ServiceManager最终逻辑的进程直接就在native层实现了，因此也不需要将逻辑传回java层再做处理。 具体的文件是 /frameworks/native/cmds/servicemanager/service_manager.c。

有兴趣的读者可以自行探索，或者关注笔者后面的文章。

# 总结
1. IActivityManger的内容在ActivityManagerService中实现，IServieManger的逻辑再ServiceManagerNative中实现。1. client进程去获取IActivityManger时，实际上是先获取的IServieManger，然后通过IServieManger的getService方法来获取IActivityManger。1. ServiceManagerNative.onTransact不会被调用，在ServiceManagerProxy中调用了IBinder.transact之后，其逻辑直接在native层处理了。
# 继续探索

本文讲解完还有很多知识点并没有讲解:
- native层是如何实现ServiceManager的逻辑的？- native层除了/frameworks/native/cmds/servicemanager/service_manager.c，还有/frameworks/native/libs/binder/IServiceManager.cpp,这类的作用分别是什么。- ActivityManagerService是什么时候执行addService逻辑的？
有兴趣的读者可以自行探索，或者关注笔者后续文章。