#由浅入深 学习 Android Binder（八）- IServiceManager与BpServiceManager（native层）
>  
 Android Binder系列文章：            


# 概述

前文已经讲解过java层ServiceManager的逻辑，如果没有了解过的读者建议看下： 

此文承接前文，继续讲下在native层的IServiceManager，具体的path是 /frameworks/native/libs/binder/IServiceManager.cpp。

>  
 native层还有/frameworks/native/cmds/servicemanager/service_manager.c ， 关于这个类，此文不会涉及，后面会专门用一篇文章讲。 


本文还是根据笔者的认知来讲一些自己认为重要的地方。 不会将一个文件的源码从头到尾的分析，个人认为那样对于新人非常不友好。 如果对一些细节的知识点有兴趣的读者，在了解了大概逻辑后完全可以再自行探索。

为了方便读者有个整体的认知，附上整体的流程图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040058700.png" alt="在这里插入图片描述">

# 寻找一个看源码的起点

全局搜索下”#include &lt;binder/IServiceManager.h&gt;“,找到引用IServiceManager类的地方,看下是怎么使用的。 笔者这边定位到的是/frameworks/av/media/libmediaplayerservice/ActivityManager.cpp这个类。 具体这个类的作用暂时先不看。我们只需要看下它是如何使用IServiceManager的：
1. 通过defaultServiceManager()获取到IServiceManager的强引用1. 调用IServiceManager的getService方法获取到IBinder对象。1. 通过binder获取到IActivityManager对象。
>  
 如果看过java层ServiceManager逻辑的读者，会发现其实native层的这个IServiceManager逻辑和java层其实差不多。 


```
int openContentProviderFile(const String16&amp; uri)
{
    int fd = -1;

    sp&lt;IServiceManager&gt; sm = defaultServiceManager();
    sp&lt;IBinder&gt; binder = sm-&gt;getService(String16("activity"));
    sp&lt;IActivityManager&gt; am = interface_cast&lt;IActivityManager&gt;(binder);
    if (am != NULL) {
        fd = am-&gt;openContentUri(uri);
    }

    return fd;
}

```

至此就有两个我们需要分析的重点了：
- IServiceManager.defaultServiceManager()- IServiceManager.getService()
>  
 至于addService等其他调用，由于实现与getService差不多，笔者就不重复解析了，有兴趣的读者可以自己探索下。 


# IServiceManager.defaultServiceManagr

```
sp&lt;IServiceManager&gt; defaultServiceManager()
{
    if (gDefaultServiceManager != NULL) return gDefaultServiceManager;

    {
        AutoMutex _l(gDefaultServiceManagerLock);
        while (gDefaultServiceManager == NULL) {
            gDefaultServiceManager = interface_cast&lt;IServiceManager&gt;(
                ProcessState::self()-&gt;getContextObject(NULL));
            if (gDefaultServiceManager == NULL)
                sleep(1);
        }
    }

    return gDefaultServiceManager;
}

```

由defaultServiceManager这个方法的逻辑可知，IServiceManager是一个单例。 那么接下来需要分析的充电其实就是这行代码：

```
gDefaultServiceManager = interface_cast&lt;IServiceManager&gt;(
                ProcessState::self()-&gt;getContextObject(NULL));

```

主要分为两部分：
- ProcessState::self()-&gt;getContextObject(NULL)- interface_cast()
### ProcessState.getContextObject(null)

```
sp&lt;IBinder&gt; ProcessState::getContextObject(const sp&lt;IBinder&gt;&amp; /*caller*/)
{
    return getStrongProxyForHandle(0);
}

```

这里最终其实就是调用到了getStrongProxyForHandle()的逻辑。

由于前文已经探索过getStrongProxyForHandle的逻辑，此处就不重复讲解了，直接得出结论，此处返回的是IServiceManager对应的handle=0的BpBinder。

>  
 对getStrongProxyForHandle的具体逻辑有兴趣的读者可以看下笔者的前文：  


### interface_cast()

interface_cast定义与这个文件中： /frameworks/native/include/binder/IInterface.h

代码如下：

```
template&lt;typename INTERFACE&gt;
inline sp&lt;INTERFACE&gt; interface_cast(const sp&lt;IBinder&gt;&amp; obj)
{
    return INTERFACE::asInterface(obj);
}

```

于是我们就需要再看下asInterface的定义。

在IServiceManager.h中有这样一段定义：

```
class IServiceManager : public IInterface
{
public:
    DECLARE_META_INTERFACE(ServiceManager)
——————————————————————省略
}

```

DECLARE_META_INTERFACE定义在IInterface中：

```
#define DECLARE_META_INTERFACE(INTERFACE)                               \
    static const ::android::String16 descriptor;                        \
    static ::android::sp&lt;I##INTERFACE&gt; asInterface(                     \
            const ::android::sp&lt;::android::IBinder&gt;&amp; obj);              \
    virtual const ::android::String16&amp; getInterfaceDescriptor() const;  \
    I##INTERFACE();                                                     \
    virtual ~I##INTERFACE();                                            \

```

将DECLARE_META_INTERFACE(ServiceManager)这段代码，根据define中的定义带入ServiceManager得到如下代码：

```
static const android::String16 descriptor;

static android::sp&lt; IServiceManager &gt; asInterface(const android::sp&lt;android::IBinder&gt;&amp; obj)

virtual const android::String16&amp; getInterfaceDescriptor() const;

IServiceManager ();
virtual ~IServiceManager();

```

这里只是定义了asInterface，我们还需要看下asInterface实现的地方。

在IServiceManager.cpp中有这样一段代码：

```
IMPLEMENT_META_INTERFACE(ServiceManager, "android.os.IServiceManager");

```

IMPLEMENT_META_INTERFACE定义在IInterface中：

```
#define IMPLEMENT_META_INTERFACE(INTERFACE, NAME)                       \
    const ::android::String16 I##INTERFACE::descriptor(NAME);           \
    const ::android::String16&amp;                                          \
            I##INTERFACE::getInterfaceDescriptor() const {              \
        return I##INTERFACE::descriptor;                                \
    }                                                                   \
    ::android::sp&lt;I##INTERFACE&gt; I##INTERFACE::asInterface(              \
            const ::android::sp&lt;::android::IBinder&gt;&amp; obj)               \
    {                                                                   \
        ::android::sp&lt;I##INTERFACE&gt; intr;                               \
        if (obj != NULL) {                                              \
            intr = static_cast&lt;I##INTERFACE*&gt;(                          \
                obj-&gt;queryLocalInterface(                               \
                        I##INTERFACE::descriptor).get());               \
            if (intr == NULL) {                                         \
                intr = new Bp##INTERFACE(obj);                          \
            }                                                           \
        }                                                               \
        return intr;                                                    \
    }                                                                   \
    I##INTERFACE::I##INTERFACE() { }                                    \
    I##INTERFACE::~I##INTERFACE() { }                                   \

```

将IMPLEMENT_META_INTERFACE(ServiceManager, “android.os.IServiceManager”)这段代码，根据define中的定义带入参数得到如下代码：

```
const android::String16 IServiceManager::descriptor(“android.os.IServiceManager”);

const android::String16&amp; IServiceManager::getInterfaceDescriptor() const
{
     return IServiceManager::descriptor;
}

 android::sp&lt;IServiceManager&gt; IServiceManager::asInterface(const android::sp&lt;android::IBinder&gt;&amp; obj)
{
       android::sp&lt;IServiceManager&gt; intr;
        if(obj != NULL) {
           intr = static_cast&lt;IServiceManager *&gt;(
               obj-&gt;queryLocalInterface(IServiceManager::descriptor).get());
           if (intr == NULL) {
               intr = new BpServiceManager(obj);  
            }
        }
       return intr;
}

IServiceManager::IServiceManager () { }
IServiceManager::~ IServiceManager() { }

```

于是我们就得到了asInterface的实现内容：
1. 先通过IBinder.queryLocalInterface带入参数descriptor，在本地进程查找IServiceManager。1. 如果IServiceManager在其他进程，那么就新建一个BpServiceManager返回。
接下来我们继续看下BpServiceManager这个对象。

>  
 如果已经了解java层aidl中的asInterface源码的读者，此处应该能发现两者逻辑其实是一样的。后续逻辑大概就能猜到是什么了。 对aidl相关内容有兴趣的读者可以看笔者前文：  


# BpServiceManager初始化

>  
 BpServiceManager就先只看下初始化，getService()的源码后面会一起讨论。 


```
    explicit BpServiceManager(const sp&lt;IBinder&gt;&amp; impl)
        : BpInterface&lt;IServiceManager&gt;(impl)
    {
    }

```

BpServiceManager初始化没有自己的内容，只是调用了父类BpInterface的构造。

BpInterface的构造定义再IInterface.cpp这个问价那种

```
template&lt;typename INTERFACE&gt;
inline BpInterface&lt;INTERFACE&gt;::BpInterface(const sp&lt;IBinder&gt;&amp; remote)
    : BpRefBase(remote)
{
}

```

BpInterface初始化没有自己的内容，只是调用了父类BpRefBase的构造。

BpRefBase定义在/frameworks/native/include/binder/Binder.cpp中,这个构造的源码如下：

```
BpRefBase::BpRefBase(const sp&lt;IBinder&gt;&amp; o)
    : mRemote(o.get()), mRefs(NULL), mState(0)
{
    extendObjectLifetime(OBJECT_LIFETIME_WEAK);

    if (mRemote) {
        mRemote-&gt;incStrong(this);           // Removed on first IncStrong().
        mRefs = mRemote-&gt;createWeak(this);  // Held for our entire lifetime.
    }
}

```

根据源码可知，BpRefBase的内容，即BpServiceManager初始化的逻辑：
1. 让mRemote持有该IBinder的强引用1. 让mRefs持有该IBinder的弱引用1. 此处传进来的IBinder就是前面解析的ProcessState.getContextObject(null)，实际上对应的是handle=0的BpBinder.
# IServiceManager.getService

根据前面对defaultServiceManager方法的分析，在其他进程调用的IServiceManager的情况下，defaultServiceManager返回的会是BpServiceManager。 因此我们实际上需要看下BpServiceManager.getService()。

>  
 此处可能有读者会好奇，如果是本进程的情况呢？ 笔者认为没有。 一方面，笔者没有见过相关使用和源码，另一方面因为ServiceManager本身就是提供给其他进程管理服务用的，本进程操作没有什么意义。 此处如果是笔者认知错误，欢迎评论交流。 


```
    virtual sp&lt;IBinder&gt; getService(const String16&amp; name) const
    {
        sp&lt;IBinder&gt; svc = checkService(name);
        if (svc != NULL) return svc;

        const bool isVendorService =
            strcmp(ProcessState::self()-&gt;getDriverName().c_str(), "/dev/vndbinder") == 0;
        const long timeout = uptimeMillis() + 5000;
        if (!gSystemBootCompleted) {
            char bootCompleted[PROPERTY_VALUE_MAX];
            property_get("sys.boot_completed", bootCompleted, "0");
            gSystemBootCompleted = strcmp(bootCompleted, "1") == 0 ? true : false;
        }
        // retry interval in millisecond.
        const long sleepTime = gSystemBootCompleted ? 1000 : 100;

        int n = 0;
        while (uptimeMillis() &lt; timeout) {
            n++;
            if (isVendorService) {
                ALOGI("Waiting for vendor service %s...", String8(name).string());
                CallStack stack(LOG_TAG);
            } else if (n%10 == 0) {
                ALOGI("Waiting for service %s...", String8(name).string());
            }
            usleep(1000*sleepTime);

            sp&lt;IBinder&gt; svc = checkService(name);
            if (svc != NULL) return svc;
        }
        ALOGW("Service %s didn't start. Returning NULL", String8(name).string());
        return NULL;
    }

```

getService源码内容如下：
1. 先通过checkService获取到IBinder的强引用，如果不为空就直接返回。1. 如果获取不到，那么在超时之前都会进入一个循环：先sleep,再获取，直至获取到位置。
其实主要就是checkService的逻辑，我们看下它的源码：

```
    virtual sp&lt;IBinder&gt; checkService( const String16&amp; name) const
    {
        Parcel data, reply;
        data.writeInterfaceToken(IServiceManager::getInterfaceDescriptor());
        data.writeString16(name);
        remote()-&gt;transact(CHECK_SERVICE_TRANSACTION, data, &amp;reply);
        return reply.readStrongBinder();
    }

```

checkService的逻辑如下：
1. 先将数据写入Parcel类型的data1. 通过IBinder.transact调用IPC方法。name是CHECK_SERVICE_TRANSACTION。1. 返回数据会写入Parcel类型的replay，最终从replay中取出IBinder返回。
>  
 transact的流程前文已经讲过，此处就不重复解析了，有兴趣的读者可以看下前文：  


# java层与native层的IServiceManager

根据上文的解析可知，native层的IServiceManager的功能和java层的IServiceManager的功能是差不多的。

>  
 还不了解java层IServiceManager的读者可以看下前文：  


实际上两者IBinder与获取的位置也是同一个。(存储位置是service_manager的list，本文不拓展) 简单来说，一个IBinder，可以使用java层的IServiceManager来注册，然后使用native层的IServiceManager来获取。反过来也可以。

还是看个例子来证明这一点：

>  
 IActivitManager所对应的name是"activity" 注册：Java层 ActivityManagerService.setSystemProcess() 调用：native层 ActivityManager.openContentProviderFile() 


### ActivityManagerService.setSystemProcess()

```
public void setSystemProcess() {<!-- -->
————————————————省略
            ServiceManager.addService(Context.ACTIVITY_SERVICE, this, /* allowIsolated= */ true,
                    DUMP_FLAG_PRIORITY_CRITICAL | DUMP_FLAG_PRIORITY_NORMAL | DUMP_FLAG_PROTO);
————————————————省略
}

```

ACTIVITY_SERVICE定义与Context中：

```
    public static final String ACTIVITY_SERVICE = "activity";

```

### ActivityManager.openContentProviderFile()

```
int openContentProviderFile(const String16&amp; uri)
{
    int fd = -1;

    sp&lt;IServiceManager&gt; sm = defaultServiceManager();
    sp&lt;IBinder&gt; binder = sm-&gt;getService(String16("activity"));
    sp&lt;IActivityManager&gt; am = interface_cast&lt;IActivityManager&gt;(binder);
    if (am != NULL) {
        fd = am-&gt;openContentUri(uri);
    }

    return fd;
}

```

# 总结
1. IServiceManager.defaultServiceManagr实际返回的是BpServiceManager对象1. BpServiceManager中的remote是handle=0的BpBinder。（其实handle=0的情况在 binder driver中其实就对应着service_manager）1. getService方法实际上就是调用的BpServiceManager的getService方法，最终会通过调用IBinder.transact方法来操作binder dirver。1. java层与native层的IServiceManager功能类似，注册于获取IBinder的位置是同一个。(存储位置是service_manager的list，本文不拓展)
>  
 至于addService等其他调用，由于实现与getService差不多，笔者就不重复解析了，有兴趣的读者可以自己探索下。 


# 继续探索

本文讲解完还有很多知识点并没有讲解:
- BpServiceManager调用getService后，是谁来处理的后续逻辑？- /frameworks/native/cmds/servicemanager/service_manager.c 这个类的逻辑是什么？
有兴趣的读者可以自行探索，或者关注笔者后续文章。