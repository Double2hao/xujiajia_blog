#Andromeda 源码解析 （同步获取服务）
# Andromeda

Andromeda是爱奇艺开源的适用于多进程架构的组件通信框架。

>  
 github地址： https://github.com/iqiyi/Andromeda 


其特点如下：
- 无需开发者进行bindService()操作,也不用定义Service,只需要定义aidl接口和实现- 同步获取服务。抛弃了bindService()这种异步获取的方式，改造成了同步获取- 生命周期自动管理。可根据Fragment或Activity的生命周期进行提高或降低服务提供方进程的操作- 支持IPC的Callback，并且支持跨进程的事件总线- 采用"接口+数据结构"的方式来实现组件间通信，这种方式相比协议的方式在于实现简单，维护方便
# 概述

众所周知，如果使用binder来实现进程间通信，我们一般需要通过自定义ServiceConnection，在onServiceConnected方法中实现通信逻辑。 如果说，在进程间通信之后需要有其他操作，那么也只能异步来实现。

而Andromeda则实现了远程服务的同步调用。 此文主要探究下Andromeda 中“远程服务的同步调用”式如何实现的。

# 基本概念

Andromeda架构的核心就Dispatcher和RemoteTransfer。 每个进程通过RemoteTransfer 来和Dispatcher进程间通信，实现业务间多进程通信的需求。

### DIspatcher

**全局只有一个**，存在于生命周期最长的进程中。（不一定是主进程，例如音乐类app，生命周期最长的就是播放音乐的进程） 负责管理所有进程的业务binder 以及 各进程中RemoteTransfer的binder。

### RemoteTransfer

**每个进程一个**。 负责管理它所在进程所有业务的binder。

### 例图

举个简单的例子，比如现在有A,B两个进程，A和B要实现进程间通信的需求，那么通信的图就如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1070.png" alt="在这里插入图片描述">

# 远程服务的同步调用

我们直接看下Andromeda项目中的demo代码：

```
    private void useBuyAppleInShop() {<!-- -->
        //IBinder buyAppleBinder = Andromeda.getInstance().getRemoteService(IBuyApple.class);
        IBinder buyAppleBinder = Andromeda.with(this).getRemoteService(IBuyApple.class);
        if (null == buyAppleBinder) {<!-- -->
            Toast.makeText(this, "buyAppleBinder is null! May be the service has been cancelled!", Toast.LENGTH_SHORT).show();
            return;
        }
        IBuyApple buyApple = IBuyApple.Stub.asInterface(buyAppleBinder);
        if (null != buyApple) {<!-- -->
            try {<!-- -->
                int appleNum = buyApple.buyAppleInShop(10);
                Toast.makeText(BananaActivity.this, "got remote service in other process(:banana),appleNum:" + appleNum, Toast.LENGTH_SHORT).show();

            } catch (RemoteException ex) {<!-- -->
                ex.printStackTrace();
            }
        }
    }

```

上面代码内容：
1. 通过Andromeda直接获取到事先注册的IBinder对象。1. 通过IBinder对象获取到IBuyApple对象。（也就是aidl文件中的对象，用于进程间通信）1. 直接通过调用IBuyApple对象中的方法实现跨进程调用。
# 源码解析

上面代码中通过getRemoteService来获取的IBinder，首先进入这个方法。

### RemoteManager.getRemoteService()

最终我们可以看到IBinder对象是从BinderBean对象中拿到的，而BinderBean对象是通过RemoteTransfer.getInstance().getRemoteServiceBean()获取到的。

```
    @Override
    public IBinder getRemoteService(Class&lt;?&gt; serviceClass) {<!-- -->
        if (null == serviceClass) {<!-- -->
            return null;
        }
        return getRemoteService(serviceClass.getCanonicalName());
    }

    @Override
    public synchronized IBinder getRemoteService(String serviceCanonicalName) {<!-- -->
        Logger.d(this.toString() + "--&gt;getRemoteService,serviceName:" + serviceCanonicalName);
        if (TextUtils.isEmpty(serviceCanonicalName)) {<!-- -->
            return null;
        }
        BinderBean binderBean = RemoteTransfer.getInstance().getRemoteServiceBean(serviceCanonicalName);
        if (binderBean == null) {<!-- -->
            Logger.e("Found no binder for "+serviceCanonicalName+"! Please check you have register implementation for it or proguard reasons!");
            return null;
        }
        String commuStubServiceName = ConnectionManager.getInstance().bindAction(appContext, binderBean.getProcessName());
        commuStubServiceNames.add(commuStubServiceName);
        return binderBean.getBinder();
    }

```

### RemoteTransfer.getRemoteServiceBean()

这里就会有两种情况：
1. 如果之前已经获取过BinderBean对象，那么可以直接从cache中获得。1. 如果之前没有获取过BinderBean对象，那么cache中不会获取到，需要通过serviceTransfer.getAndSaveIBinder()这个方法从Dispatcher中获取。
接下来看下serviceTransfer.getIBinderFromCache()和serviceTransfer.getAndSaveIBinder()这两个方法。

```
    @Override
    public synchronized BinderBean getRemoteServiceBean(String serviceCanonicalName) {<!-- -->
        Logger.d("RemoteTransfer--&gt;getRemoteServiceBean,pid=" + android.os.Process.myPid() + ",thread:" + Thread.currentThread().getName());
        BinderBean cacheBinderBean = serviceTransfer.getIBinderFromCache(context, serviceCanonicalName);
        if (cacheBinderBean != null) {<!-- -->
            return cacheBinderBean;
        }
        initDispatchProxyLocked();
        if (serviceTransfer == null || dispatcherProxy == null) {<!-- -->
            return null;
        }
        return serviceTransfer.getAndSaveIBinder(serviceCanonicalName, dispatcherProxy);
    }

```

### serviceTransfer.getIBinderFromCache()

首先看下是否是本地服务，如果是本地服务，就不进行bind操作，直接封装一个BinderBean返回。 如果不是本地服务，那么就查看下缓存中是否有该对象，有的话就返回。

```
    public BinderBean getIBinderFromCache(Context context, String serviceCanonicalName) {<!-- -->
        //如果是自己进程或者主进程，就不要进行bind操作了
        if (stubBinderCache.get(serviceCanonicalName) != null) {<!-- -->
            return new BinderBean(stubBinderCache.get(serviceCanonicalName),
                    ProcessUtils.getProcessName(context));
        }

        if (remoteBinderCache.get(serviceCanonicalName) != null) {<!-- -->
            return remoteBinderCache.get(serviceCanonicalName);
        }
        return null;
    }

```

### serviceTransfer.getAndSaveIBinder()

通过IDispatcher对象来获取BinderBean对象，如果获取到了，那么久放到cache中。 同时，为BinderBean对象中的Binder设置death的监听，如果binder意外死亡了，那么就要从cache中移除。

```
    public BinderBean getAndSaveIBinder(final String serviceName, IDispatcher dispatcherProxy) {<!-- -->
        try {<!-- -->
            BinderBean binderBean = dispatcherProxy.getTargetBinder(serviceName);
            if (null == binderBean) {<!-- -->
                return null;
            }
            try {<!-- -->
                binderBean.getBinder().linkToDeath(new IBinder.DeathRecipient() {<!-- -->
                    @Override
                    public void binderDied() {<!-- -->
                        remoteBinderCache.remove(serviceName);
                    }
                }, 0);
            } catch (RemoteException ex) {<!-- -->
                ex.printStackTrace();
            }
            Logger.d("get IBinder from ServiceDispatcher");
            remoteBinderCache.put(serviceName, binderBean);
            return binderBean;
        } catch (RemoteException ex) {<!-- -->
            ex.printStackTrace();
        }
        return null;
    }

```