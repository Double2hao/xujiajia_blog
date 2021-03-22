#由浅入深 学习 Android Binder（三）- java binder深究（从java到native）
>  
 Android Binder系列文章：           

>[由浅入深 学习 Android Binder（一）- AIDL](https://xujiajia.blog.csdn.net/article/details/109865496)

>[由浅入深 学习 Android Binder（二）- bindService流程](https://xujiajia.blog.csdn.net/article/details/109906012)

>[由浅入深 学习 Android Binder（三）- java binder深究（从java到native）](https://xujiajia.blog.csdn.net/article/details/110730526)

>[由浅入深 学习 Android Binder（四）- ibinderForJavaObject 与 javaObjectForIBinder](https://xujiajia.blog.csdn.net/article/details/111027972)

>[由浅入深 学习 Android Binder（五）- binder如何在进程间传递](https://xujiajia.blog.csdn.net/article/details/111057369)

>[由浅入深 学习 Android Binder（六）- IPC 调用流程](https://xujiajia.blog.csdn.net/article/details/111399789)

>[由浅入深 学习 Android Binder（七）- IServiceManager与ServiceManagerNative（java层）](https://xujiajia.blog.csdn.net/article/details/112131416)

>[由浅入深 学习 Android Binder（八）- IServiceManager与BpServiceManager（native层）](https://xujiajia.blog.csdn.net/article/details/112131416)

>[由浅入深 学习 Android Binder（九）- service_manager 与 svclist](https://xujiajia.blog.csdn.net/article/details/112733698)

>[由浅入深 学习 Android Binder（十）- 总结](https://xujiajia.blog.csdn.net/article/details/112733857)

>[由浅入深 学习 Android Binder（十一) binder线程池](https://xujiajia.blog.csdn.net/article/details/115054785)


# 概述

>  
 前文地址： 


前文讲到bindService流程，其中多次碰到用binder实现的IPC，有以下几个：
1. client进程 通过IPC 通知AMS进程去bindService
2. AMS进程 通过IPC 通知server进程去bindService
3. server进程 通过IPC 通知AMS进程bindService结果
4. AMS进程 通过IPC 告诉client进程bindService结果
本文就选择”client通知AMS进程“的过程继续作深入，来探究下java层的binder究竟是什么。(附上前文流程图)

<img src="https://img-blog.csdnimg.cn/20201206051130845.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# ContextImpl.bindServiceCommon()

client进程 通知IPC 去bindService,最终会走到ContextImpl. bindServiceCommon().

```
private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags, Handler
            handler, UserHandle user) {<!-- -->
        // Keep this in sync with DevicePolicyManager.bindDeviceAdminServiceAsUser.
        IServiceConnection sd;
        if (conn == null) {<!-- -->
            throw new IllegalArgumentException("connection is null");
        }
        if (mPackageInfo != null) {<!-- -->
            sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(), handler, flags);
        } else {<!-- -->
            throw new RuntimeException("Not supported in system context");
        }
        validateServiceIntent(service);
        try {<!-- -->
            IBinder token = getActivityToken();
            if (token == null &amp;&amp; (flags&amp;BIND_AUTO_CREATE) == 0 &amp;&amp; mPackageInfo != null
                    &amp;&amp; mPackageInfo.getApplicationInfo().targetSdkVersion
                    &lt; android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH) {<!-- -->
                flags |= BIND_WAIVE_PRIORITY;
            }
            service.prepareToLeaveProcess(this);
            int res = ActivityManager.getService().bindService(
                mMainThread.getApplicationThread(), getActivityToken(), service,
                service.resolveTypeIfNeeded(getContentResolver()),
                sd, flags, getOpPackageName(), user.getIdentifier());
            if (res &lt; 0) {<!-- -->
                throw new SecurityException(
                        "Not allowed to bind to service " + service);
            }
            return res != 0;
        } catch (RemoteException e) {<!-- -->
            throw e.rethrowFromSystemServer();
        }
    }

```

我们只需要关注其中一行代码即可。

```
            int res = ActivityManager.getService().bindService(
                mMainThread.getApplicationThread(), getActivityToken(), service,
                service.resolveTypeIfNeeded(getContentResolver()),
                sd, flags, getOpPackageName(), user.getIdentifier());

```

接着看下ActivityManager.getService()的逻辑：

# ActivityManager.getService()

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

代码看到这，就可以确定是使用binder来实现的IPC了。ActivityManager.getService()实际返回的是IActivityManager。 又因为源码中只有ActivityManagerService实现了IActivityManager.Stub，因此，此处可以确定返回的是ActivityManagerService对象。

此处继续深入，看下这个IActivityManager对象究竟是从哪里获取到的。 于是我们看下ServiceManager.getService()逻辑。

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

既然是深究，那么此处缓存的逻辑也可以暂且不用看了。 直接进入到rawGetService（）.

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
1. 获取到IServiceManager
2. 通过IServiceManager获取到该name的IPC

由于本文本文重在探索java到native的逻辑，我们且只看第一个。

>  
 关于第二个： 源码中实现IServiceManager的类是ServiceManagerNative 有兴趣的读者可以自己探索下，或者关注笔者后面的文章 


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

到这，可以看到返回的其实是ServiceManagerNative.asInterface()的返回值。

对AIDL有所了解就知道，asInterface()的内容大概如下：
- 这个方法属于aidl接口的内部类 Stub
- 在同一进程中，就会直接返回Stub，如果在另一个进程中调用，就会返回将这个ibinder封装好的Proxy对象。
>  
 对aidl不了解的可以看下前面的文章： 


于是，虽然没有看ServiceManagerNative的源码，但是其逻辑已经很清晰了。 我们只需要关注IBinder的来源就可以了，也就是BinderInternal.getContextObject()。

# BinderInternal.getContextObject()

```
    /**
     * Return the global "context object" of the system.  This is usually
     * an implementation of IServiceManager, which you can use to find
     * other services.
     */
    public static final native IBinder getContextObject();

```

至此，可以发现IBinder对象其实是从native层获取到的。 我们继续看下JNI的代码。

# android_util_Binder.android_os_BinderInternal_getContextObject()

```
static jobject android_os_BinderInternal_getContextObject(JNIEnv* env, jobject clazz)
{
    sp&lt;IBinder&gt; b = ProcessState::self()-&gt;getContextObject(NULL);
    return javaObjectForIBinder(env, b);
}

```

这里先通过ProcessState获取到了一个native 层的IBinder强引用。 然后将这个native层的IBinder强引用传入javaObjectForIBinder()方法，最终封装成java层的IBinder然后返回。

此处先不深究ProcessState的逻辑，整个native层的binder有自己的一整套的逻辑，后面的文章会继续探索。

我们可以先稍微看下javaObjectForIBinder()的大概逻辑。

# android_util_Binder.javaObjectForIBinder()

```
// If the argument is a JavaBBinder, return the Java object that was used to create it.
// Otherwise return a BinderProxy for the IBinder. If a previous call was passed the
// same IBinder, and the original BinderProxy is still alive, return the same BinderProxy.
jobject javaObjectForIBinder(JNIEnv* env, const sp&lt;IBinder&gt;&amp; val)
{
    if (val == NULL) return NULL;

    if (val-&gt;checkSubclass(&amp;gBinderOffsets)) {
        // It's a JavaBBinder created by ibinderForJavaObject. Already has Java object.
        jobject object = static_cast&lt;JavaBBinder*&gt;(val.get())-&gt;object();
        LOGDEATH("objectForBinder %p: it's our own %p!\n", val.get(), object);
        return object;
    }

    // For the rest of the function we will hold this lock, to serialize
    // looking/creation/destruction of Java proxies for native Binder proxies.
    AutoMutex _l(gProxyLock);

    BinderProxyNativeData* nativeData = gNativeDataCache;
    if (nativeData == nullptr) {
        nativeData = new BinderProxyNativeData();
    }
    // gNativeDataCache is now logically empty.
    jobject object = env-&gt;CallStaticObjectMethod(gBinderProxyOffsets.mClass,
            gBinderProxyOffsets.mGetInstance, (jlong) nativeData, (jlong) val.get());
    if (env-&gt;ExceptionCheck()) {
        // In the exception case, getInstance still took ownership of nativeData.
        gNativeDataCache = nullptr;
        return NULL;
    }
    BinderProxyNativeData* actualNativeData = getBPNativeData(env, object);
    if (actualNativeData == nativeData) {
        // New BinderProxy; we still have exclusive access.
        nativeData-&gt;mOrgue = new DeathRecipientList;
        nativeData-&gt;mObject = val;
        gNativeDataCache = nullptr;
        ++gNumProxies;
        if (gNumProxies &gt;= gProxiesWarned + PROXY_WARN_INTERVAL) {
            ALOGW("Unexpectedly many live BinderProxies: %d\n", gNumProxies);
            gProxiesWarned = gNumProxies;
        }
    } else {
        // nativeData wasn't used. Reuse it the next time.
        gNativeDataCache = nativeData;
    }

    return object;
}

```

```
class JavaBBinder : public BBinder
{
public:
——————————————————省略
    jobject object() const
    {
        return mObject;
    }
——————————————————省略
private:
    JavaVM* const   mVM;
    jobject const   mObject;  // GlobalRef to Java Binder
};

```

这个方法中未知的细节很多，但是大致逻辑其实很简单，对具体细节没兴趣的读者直接看注释就可以。 这个方法会返回两种java对象：
-  JavaBBinder.mObject 如果发现本地已经有了JavaBBinder，就返回JavaBBinder.mObject。这个object其实是java层的Binder。 
-  BinderProxy 如果发现本地没有BinderProxy，就会调用java层的方法创建一个BinderProxy并返回。（BinderProxy是java层的android.os.BinderProxy） 
>  
 至于这些对象在分别代表什么，讨论起来需要很大的篇幅。笔者后面的文章会继续探索。 


至此，我们已经大概了解了java 层的binder。 这个binder对象有可能是Binder，也有可能是BinderProxy。它与native的逻辑息息相关。

# 总结

本文从bindService 其中的一个IPC出发，一步步深究到native层。 旨在能给读者一个新的思路去理解binder，也是从java到native层分析的一个过渡。

# 继续探索

这篇文章后肯定还有很多疑问，比如:
- JavaBBinder和BinderProxy具体是啥？
- 到底什么场景用哪个对象？- native层主要做了啥?

对这些问题有兴趣的读者可以自行观看源码，或者 欢迎继续关注笔者后续文章。