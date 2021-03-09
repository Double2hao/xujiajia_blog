#由浅入深 学习 Android Binder（四）- ibinderForJavaObject 与 javaObjectForIBinder
>  
 Android Binder系列文章：           


# 概述

前文已经解析了java层binder在native层的形式。

>  
 前文地址： 


前文由于篇幅限制，没有继续探索JavaBBinder与BinderProxy，此文则将继续深入相关逻辑。

# ibinderForJavaObject 与 javaObjectForIBinder

java与native之间通过JNI来实现数据的传递与交互。 主要就是通过native层的ibinderForJavaObject与javaObjectForIBinder方法。 整个流程大概就是下面两张图，先列出便于读者有一个整理的认知.

<img src="https://img-blog.csdnimg.cn/20201211082725338.png" alt="在这里插入图片描述"> <img src="https://img-blog.csdnimg.cn/20201211082725333.png" alt="在这里插入图片描述">

# android_util_Binder.ibinderForJavaObject

逻辑如下：
1. 如果这个java对象是 java Binder对象，那么先获取到本地的JavaBBinderHolder对象，然后从JavaBBinderHolder中get到IBinder对象返回。1. 如果这个java对象是 java BinderProxy对象，那么通过getBPNativeData()方法获取到某个对象后返回其mObject。
接着我们看下getBPNativeData这个方法。

```
sp&lt;IBinder&gt; ibinderForJavaObject(JNIEnv* env, jobject obj)
{
    if (obj == NULL) return NULL;

    // Instance of Binder?
    if (env-&gt;IsInstanceOf(obj, gBinderOffsets.mClass)) {
        JavaBBinderHolder* jbh = (JavaBBinderHolder*)
            env-&gt;GetLongField(obj, gBinderOffsets.mObject);
        return jbh-&gt;get(env, obj);
    }

    // Instance of BinderProxy?
    if (env-&gt;IsInstanceOf(obj, gBinderProxyOffsets.mClass)) {
        return getBPNativeData(env, obj)-&gt;mObject;
    }

    ALOGW("ibinderForJavaObject: %p is not a Binder object", obj);
    return NULL;
}

```

### android_util_Binder.getBPNativeData

最终返回的其实是一个BinderProxyNativeData，而mObject则是其中存储的mObject对象。 通过注释可知，是在调用javaObjectForIBinder方法的时候创建的。

于是我们就直接看下javaObjectForIBinder代码

```

BinderProxyNativeData* getBPNativeData(JNIEnv* env, jobject obj) {
    return (BinderProxyNativeData *) env-&gt;GetLongField(obj, gBinderProxyOffsets.mNativeData);
}

struct BinderProxyNativeData {
    // Both fields are constant and not null once javaObjectForIBinder returns this as
    // part of a BinderProxy.
    
    // The native IBinder proxied by this BinderProxy.
    sp&lt;IBinder&gt; mObject;

    // Death recipients for mObject. Reference counted only because DeathRecipients
    // hold a weak reference that can be temporarily promoted.
    sp&lt;DeathRecipientList&gt; mOrgue;  // Death recipients for mObject.
};

```

# android_util_Binder.javaObjectForIBinder

这个类有以下要点：
1. 通过gBinderOffsets来判断是否是JavaBBinder对象。如果是就直接return。1. 如果前面的判断不是JavaBBinder对象。那么就会走到BinderProxy的逻辑中。 这里BinderProxy是通过gBinderProxyOffsets调用java层BinderProxy的初始化方法创建的，其中还传入了BinderProxyNativeData对象。 (另外还有一系列的BinderProxyNativeData对象相关的逻辑，这里读者只要知道一个BinderProxy对应一个BinderProxyNativeData对象就可以了)
此处的需要继续探索的点有以下几个：
1. checkSubclass是什么逻辑？以及gBinderOffsets是什么？1. gBinderProxyOffsets是什么结构？1. BinderProxyNativeData是什么结构？1. getBPNativeData()的逻辑是什么？
我们接下来一一分析。

```
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

### IBinder.checkSubclass()与gBinderOffsets

全局中定义了checkSubclass代码的只有两个地方，IBinder与JavaBBinder。

>  
 IBinder是BBinder的父类，BBinder是JavaBBinder的父类。 


IBinder.checkSubClass()

```
bool IBinder::checkSubclass(const void* /*subclassID*/) const
{
    return false;
}

```

JavaBBinder.checkSubClass()

```
    bool    checkSubclass(const void* subclassID) const
    {
        return subclassID == &amp;gBinderOffsets;
    }

```

于是，根据代码可知，checkSubClass这个方法，只有当这个IBinder对象是JavaBBinder，并且传进来的subclassID是gBinderOffsets的时候，才会返回true。 gBinderOffsets这个结构体也可以直接在源码中看到。

```
static struct bindernative_offsets_t
{
    // Class state.
    jclass mClass;
    jmethodID mExecTransact;

    // Object state.
    jfieldID mObject;

} gBinderOffsets;

```

在android_util_Binder中也可以找到初始化的方法。 因此bindernative_offsets_t这个结构体其实存储了以下内容：
- java层的Binder对象- java层Binder的execTransact方法- java层Binder的mObject对象
```
static int int_register_android_os_Binder(JNIEnv* env)
{
    jclass clazz = FindClassOrDie(env, kBinderPathName);

    gBinderOffsets.mClass = MakeGlobalRefOrDie(env, clazz);
    gBinderOffsets.mExecTransact = GetMethodIDOrDie(env, clazz, "execTransact", "(IJJI)Z");
    gBinderOffsets.mObject = GetFieldIDOrDie(env, clazz, "mObject", "J");

    return RegisterMethodsOrDie(
        env, kBinderPathName,
        gBinderMethods, NELEM(gBinderMethods));
}

```

### binderproxy_offsets_t 与 gBinderProxyOffsets

根据源码可知gBinderProxyOffsets这个对象存储了以下内容：
- java层的BinderProxy对象- BinderProxy的getInstance方法- BinderProxy的sendDeathNotice方法- BinderProxy的dumpProxyDebugInfo方法- BinderProxy的mNativeData对象，实际上指向的就是native层这个BinderProxy对应的BinderProxyNativeData
```
static struct binderproxy_offsets_t
{
    // Class state.
    jclass mClass;
    jmethodID mGetInstance;
    jmethodID mSendDeathNotice;
    jmethodID mDumpProxyDebugInfo;

    // Object state.
    jfieldID mNativeData;  // Field holds native pointer to BinderProxyNativeData.
} gBinderProxyOffsets;

```

```
static int int_register_android_os_BinderProxy(JNIEnv* env)
{
    jclass clazz = FindClassOrDie(env, "java/lang/Error");
    gErrorOffsets.mClass = MakeGlobalRefOrDie(env, clazz);

    clazz = FindClassOrDie(env, kBinderProxyPathName);
    gBinderProxyOffsets.mClass = MakeGlobalRefOrDie(env, clazz);
    gBinderProxyOffsets.mGetInstance = GetStaticMethodIDOrDie(env, clazz, "getInstance",
            "(JJ)Landroid/os/BinderProxy;");
    gBinderProxyOffsets.mSendDeathNotice = GetStaticMethodIDOrDie(env, clazz, "sendDeathNotice",
            "(Landroid/os/IBinder$DeathRecipient;)V");
    gBinderProxyOffsets.mDumpProxyDebugInfo = GetStaticMethodIDOrDie(env, clazz, "dumpProxyDebugInfo",
            "()V");
    gBinderProxyOffsets.mNativeData = GetFieldIDOrDie(env, clazz, "mNativeData", "J");

    clazz = FindClassOrDie(env, "java/lang/Class");
    gClassOffsets.mGetName = GetMethodIDOrDie(env, clazz, "getName", "()Ljava/lang/String;");

    return RegisterMethodsOrDie(
        env, kBinderProxyPathName,
        gBinderProxyMethods, NELEM(gBinderProxyMethods));
}

```

### android_util_Binder.BinderProxyNativeData

这个类就两个属性，根据注释有以下要点：
- 每个BinderProxy对象对应一个BinderProxyNativeData- 存储了native层对应的Binder对象- 存储了该Binder对象的死亡回调
```
// We aggregate native pointer fields for BinderProxy in a single object to allow
// management with a single NativeAllocationRegistry, and to reduce the number of JNI
// Java field accesses. This costs us some extra indirections here.
struct BinderProxyNativeData {
    // Both fields are constant and not null once javaObjectForIBinder returns this as
    // part of a BinderProxy.

    // The native IBinder proxied by this BinderProxy.
    sp&lt;IBinder&gt; mObject;

    // Death recipients for mObject. Reference counted only because DeathRecipients
    // hold a weak reference that can be temporarily promoted.
    sp&lt;DeathRecipientList&gt; mOrgue;  // Death recipients for mObject.
};

```

### android_util_Binder.getBPNativeData()

在前面了解了gBinderProxyOffsets对象之后，这边逻辑其实很简单。 这里就是把java层BinderProxy.mNativeData转化成native层的BinderProxyNativeData.

```
BinderProxyNativeData* getBPNativeData(JNIEnv* env, jobject obj) {
    return (BinderProxyNativeData *) env-&gt;GetLongField(obj, gBinderProxyOffsets.mNativeData);
}

```

# ibinderForJavaObject 与 javaObjectForIBinder总结

要点：
- java层直接与native层交互的对象有两个——Binder对象与BinderProxy对象。 Binder对应“Binder在本进程”的场景，BinderProxy对应“Binder在其他进程”的场景。- native层javaBBinder与java层的Binder一一对应。 native层的BinderProxyNativeData与java层的BinderProxy一一对应。- 在native层，gBinderProxyOffsets(binderproxy_offsets_t)存储了java层binderProxy的对象与需要调用的方法和属性。gBinderOffsets(binderproxy_offsets_t)存储了java层binder的对象与需要调用的方法和属性。- ibinderForJavaObject负责通过java的Binder或者BinderProxy对象，找到并返回native层的IBinder对象。- javaObjectForIBinder通过native层的IBinder对象，找到或者封装成java对象返回。 <img src="https://img-blog.csdnimg.cn/20201211082725338.png" alt="在这里插入图片描述"> <img src="https://img-blog.csdnimg.cn/20201211082725333.png" alt="在这里插入图片描述">