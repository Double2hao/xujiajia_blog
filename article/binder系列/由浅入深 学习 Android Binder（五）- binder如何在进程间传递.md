#由浅入深 学习 Android Binder（五）- binder如何在进程间传递
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

前文分析过bindService源码，该文章地址如下：  最终得到时序图如下： <img src="https://img-blog.csdnimg.cn/20201121202423379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70#pic_center" alt="在这里插入图片描述">

如上图，client进程与server进程是通过系统进程来进行通信的。

实际demo中，有两个场景我们会接触到binder：
1. client进程通过serviceConnection获取到系统进程传递的binder。
1. server进程将service.onbind()返回的binder传递给系统进程。

我们就以server给系统进程传递binder为例，来看下“进程间是如何传递binder”，或者说“binder是以什么形式在进程间传递”。

>  
 client进程与系统进程是通过IActivityManager来实现的，IActivityManager本身就是一个binder，怎么还可以传递binder？ 
 - IActivityManager是系统的Binder，而传递的Binder则是我们自定义的Binder。
 - client进程与server进程不允许直接通信，必须通过系统进程。client、server 主要调用系统进程的方式就是IActivityManager。
 - 通过序列化自定义的Binder，可以将自己本地进程的行为传递出去，让另一个进程可以通过这个Binder调用到对应的逻辑。 


# server进程给系统进程传递binder

我们直接看下server传递binder的相关源码。

### RemoteTestService.java

首先我们看下demo中的service代码，service会在onBind方法中返回Binder对象。

```
public class RemoteTestService extends Service {<!-- -->

  private ITestServer.Stub serverBinder = new ITestServer.Stub() {<!-- -->
    @Override public int testFunction(String s) throws RemoteException {<!-- -->
      Log.i("test","testFunction s= " + s);
      return 0;
    }
  };

  @Nullable @Override
  public IBinder onBind(Intent intent) {<!-- -->
    return serverBinder;
  }
}

```

### ActivityThread.handleBindService()

接着直接看下调用onBind()的地方,可以找到是ActivityThread类：

```
    private void handleBindService(BindServiceData data) {<!-- -->
    
————————————————————————省略
        IBinder binder = s.onBind(data.intent);
                        ActivityManager.getService().publishService(
                                data.token, data.intent, binder);
————————————————————————省略
    }

```

### ActivityManager.getService()

根据源码可知，getService()实际返回的是IActivityManager对象。 于是我们可以确定最终执行到的是IActivityManager.publishService()方法。

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

### IActivityManager.aidl

IActivityManager实际上是通过aidl生成的一个接口类。

在publishService中传递了IBinder的对象。 那么这个IBinder对象是如何传递的呢？

由于笔者没有找到IActivityManager.aidl系统编译后的源码，因此自定义了一个aidl看下源码。

```
interface IActivityManager {<!-- -->
————————————————————————省略
    void publishService(in IBinder token, in Intent intent, in IBinder service);
————————————————————————省略
}

```

# 自定义aidl传递IBinder

我们通过自定义一个aidl文件来看下IPC方法传递IBinder最终生成的源码是什么。 如下，自定义了ITestBinder.aidl文件：

```
interface ITestBinder {<!-- -->
    int testFunction(IBinder binder);
}

```

最终生成的java文件如下：

```
public interface ITestBinder extends android.os.IInterface
{<!-- -->
  ————————————————————————省略
  public static abstract class Stub extends android.os.Binder implements com.example.bindservicetest.ITestBinder
  {<!-- -->
    ————————————————————————省略
    @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
    {<!-- -->
      java.lang.String descriptor = DESCRIPTOR;
      switch (code)
      {<!-- -->
        case INTERFACE_TRANSACTION:
        {<!-- -->
          reply.writeString(descriptor);
          return true;
        }
        case TRANSACTION_testFunction:
        {<!-- -->
          data.enforceInterface(descriptor);
          android.os.IBinder _arg0;
          _arg0 = data.readStrongBinder();
          int _result = this.testFunction(_arg0);
          reply.writeNoException();
          reply.writeInt(_result);
          return true;
        }
        default:
        {<!-- -->
          return super.onTransact(code, data, reply, flags);
        }
      }
    }
    private static class Proxy implements com.example.bindservicetest.ITestBinder
    {<!-- -->
      ————————————————————————省略
      @Override public int testFunction(android.os.IBinder binder) throws android.os.RemoteException
      {<!-- -->
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        int _result;
        try {<!-- -->
          _data.writeInterfaceToken(DESCRIPTOR);
          _data.writeStrongBinder(binder);
          boolean _status = mRemote.transact(Stub.TRANSACTION_testFunction, _data, _reply, 0);
          if (!_status &amp;&amp; getDefaultImpl() != null) {<!-- -->
            return getDefaultImpl().testFunction(binder);
          }
          _reply.readException();
          _result = _reply.readInt();
        }
        finally {<!-- -->
          _reply.recycle();
          _data.recycle();
        }
        return _result;
      }
      public static com.example.bindservicetest.ITestBinder sDefaultImpl;
    }
    ————————————————————————省略
  }

}

```

根据源码可知，最终其实就是通过android.os.Parcel来传递的：
- 通过Parcel.readStrongBinder()来获取Binder。
- 通过Parcel.writeStrongBinder(IBinder)来写入Binder。

至此，“binder如何在进程间传递”的问题其实就转化成了“Parcel读写binder的原理是什么”。

# 写Binder原理

先放下最终的流程图，便于读者能有一个整理的理解。 <img src="https://img-blog.csdnimg.cn/20201212101356997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

### (java) Parcel.writeStrongBinder(IBinder)

最终会进入下native方法nativeWriteStrongBinder

```
    public final void writeStrongBinder(IBinder val) {<!-- -->
        nativeWriteStrongBinder(mNativePtr, val);
    }
    private static native void nativeWriteStrongBinder(long nativePtr, IBinder val);

```

### android_os_Parcel.gParcelMethods

在android_os_Parcel文件的gParcelMethods对象中，我们可以看到nativeWriteStrongBinder对应的方法是android_os_Parcel_writeStrongBinder。

```
static const JNINativeMethod gParcelMethods[] = {
————————————————————————省略
    {"nativeWriteStrongBinder",   "(JLandroid/os/IBinder;)V", (void*)android_os_Parcel_writeStrongBinder},
————————————————————————省略
};

```

### android_os_Parcel.android_os_Parcel_writeStrongBinder

逻辑如下：
1. 先通过nativePtr创建native层的Parcel对象。
1. 通过ibinderForJavaObject创建binder
1. 通过Parcel.writeStrongBinder将binder写入

>  
 ibinderForJavaObject的逻辑在前文已经讲解过，不了解的读者可以看下前文： 


```
static void android_os_Parcel_writeStrongBinder(JNIEnv* env, jclass clazz, jlong nativePtr, jobject object)
{
    Parcel* parcel = reinterpret_cast&lt;Parcel*&gt;(nativePtr);
    if (parcel != NULL) {
        const status_t err = parcel-&gt;writeStrongBinder(ibinderForJavaObject(env, object));
        if (err != NO_ERROR) {
            signalExceptionForError(env, clazz, err);
        }
    }
}

```

### Parcel.writeStrongBinder

调用flatten_binder方法，传入了ProcessState对象。

```
status_t Parcel::writeStrongBinder(const sp&lt;IBinder&gt;&amp; val)
{
    return flatten_binder(ProcessState::self(), val, this);
}

```

### Parcel.flatten_binder

此处且不看binder为null的异常情况，正常情况下的逻辑如下：
- 如果binder不是本地的，即不是本进程的。 flat_binder_object的hdr.type会设置成BINDER_TYPE_HANDLE。 flat_binder_object.binder设置成0，即没有值。 flat_binder_object.handle会设置成proxy的handle值。
- 如果binder是本地的，即本进程的。 flat_binder_object的hdr.type会设置成BINDER_TYPE_BINDER。 flat_binder_object.binder设置成本地本地binder的弱引用。 flat_binder_object.cookie设置成本地的binder。
- 最后通过finish_flatten_binder将binder写入到Parcel中。
>  
 finish_flatten_binder中通过调用Parcel.writeObject方法来写入flat_binder_object. 其最终的逻辑其实是给binder增加强引用计数，避免被回收。 由于此逻辑与深究“binder在进程间的传递形式”没有关系，在此就不具体分析了，有兴趣的读者可以自己看下。 


```
inline static status_t finish_flatten_binder(
    const sp&lt;IBinder&gt;&amp; /*binder*/, const flat_binder_object&amp; flat, Parcel* out)
{
    return out-&gt;writeObject(flat);
}

status_t flatten_binder(const sp&lt;ProcessState&gt;&amp; /*proc*/,
    const sp&lt;IBinder&gt;&amp; binder, Parcel* out)
{
    flat_binder_object obj;

    if (binder != NULL) {
        BHwBinder *local = binder-&gt;localBinder();
        if (!local) {
            BpHwBinder *proxy = binder-&gt;remoteBinder();
            if (proxy == NULL) {
                ALOGE("null proxy");
            }
            const int32_t handle = proxy ? proxy-&gt;handle() : 0;
            obj.hdr.type = BINDER_TYPE_HANDLE;
            obj.flags = FLAT_BINDER_FLAG_ACCEPTS_FDS;
            obj.binder = 0; /* Don't pass uninitialized stack data to a remote process */
            obj.handle = handle;
            obj.cookie = 0;
        } else {
            // Get policy and convert it
            int policy = local-&gt;getMinSchedulingPolicy();
            int priority = local-&gt;getMinSchedulingPriority();

            obj.flags = priority &amp; FLAT_BINDER_FLAG_PRIORITY_MASK;
            obj.flags |= FLAT_BINDER_FLAG_ACCEPTS_FDS | FLAT_BINDER_FLAG_INHERIT_RT;
            obj.flags |= (policy &amp; 3) &lt;&lt; FLAT_BINDER_FLAG_SCHEDPOLICY_SHIFT;
            obj.hdr.type = BINDER_TYPE_BINDER;
            obj.binder = reinterpret_cast&lt;uintptr_t&gt;(local-&gt;getWeakRefs());
            obj.cookie = reinterpret_cast&lt;uintptr_t&gt;(local);
        }
    } else {
        obj.hdr.type = BINDER_TYPE_BINDER;
        obj.binder = 0;
        obj.cookie = 0;
    }

    return finish_flatten_binder(binder, obj, out);
}

```

# 读Binder原理

先放下最终的流程图，便于读者能有一个整理的理解。
 <img src="https://img-blog.csdnimg.cn/20201212101436937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

### (java) Parcel.readStrongBinder()

最终会进入下native方法nativeReadStrongBinder。 mNativePtr指向的是native层的parcel。

```
    public final IBinder readStrongBinder() {<!-- -->
        return nativeReadStrongBinder(mNativePtr);
    }

    private static native IBinder     private static native IBinder nativeReadStrongBinder(long nativePtr);

```

### android_os_Parcel.gParcelMethods

在android_os_Parcel文件的gParcelMethods对象中，我们可以看到nativeReadStrongBinder对应的方法是android_os_Parcel_readStrongBinder。

```
static const JNINativeMethod gParcelMethods[] = {
————————————————————————省略
    {"nativeReadStrongBinder",    "(J)Landroid/os/IBinder;", (void*)android_os_Parcel_readStrongBinder},
————————————————————————省略
};

```

### android_os_Parcel.android_os_Parcel_readStrongBinder

此处通过native层的parcel找到binder对象，然后通过javaObjectForIBinder这个方法转化成java对象返回。

由于此处主要是深究下binder是啥，因此我们就直接看下Parcel的readStrongBinder方法。

>  
 javaObjectForIBinder的逻辑在前文已经讲解过，不了解的读者可以看下前文： 


```
static jobject android_os_Parcel_readStrongBinder(JNIEnv* env, jclass clazz, jlong nativePtr)
{
    Parcel* parcel = reinterpret_cast&lt;Parcel*&gt;(nativePtr);
    if (parcel != NULL) {
        return javaObjectForIBinder(env, parcel-&gt;readStrongBinder());
    }
    return NULL;
}

```

### (native) Parcel.readStrongBinder

逻辑很清晰，直接看unflatten_binder代码。

```
sp&lt;IBinder&gt; Parcel::readStrongBinder() const
{
    sp&lt;IBinder&gt; val;
    readNullableStrongBinder(&amp;val);
    return val;
}

status_t Parcel::readStrongBinder(sp&lt;IBinder&gt;* val) const
{
    status_t status = readNullableStrongBinder(val);
    if (status == OK &amp;&amp; !val-&gt;get()) {
        status = UNEXPECTED_NULL;
    }
    return status;
}

status_t Parcel::readNullableStrongBinder(sp&lt;IBinder&gt;* val) const
{
    return unflatten_binder(ProcessState::self(), *this, val);
}


```

### Parcel.unflatten_binder

根据unflatten_binder源码可知逻辑如下：
1. 通过Parcel获取到flat_binder_object对象
1. 根据flat_binder_object-&gt;hdr.type来走两种不同的逻辑
1. 如果type为BINDER_TYPE_BINDER，则会返回根据flat_binder_object-&gt;cookie。
1. 如果type为BINDER_TYPE_HANDLE,则会通过ProcessState的getStrongProxyForHandle传入handle，来获取Ibinder后返回。

finish_unflatten_binder只是会返回一个状态，因此不需要关注。

那么，flat_binder_object是什么？

```
status_t unflatten_binder(const sp&lt;ProcessState&gt;&amp; proc,
    const Parcel&amp; in, sp&lt;IBinder&gt;* out)
{
    const flat_binder_object* flat = in.readObject&lt;flat_binder_object&gt;();

    if (flat) {
        switch (flat-&gt;hdr.type) {
            case BINDER_TYPE_BINDER:
                *out = reinterpret_cast&lt;IBinder*&gt;(flat-&gt;cookie);
                return finish_unflatten_binder(NULL, *flat, in);
            case BINDER_TYPE_HANDLE:
                *out = proc-&gt;getStrongProxyForHandle(flat-&gt;handle);
                return finish_unflatten_binder(
                    static_cast&lt;BpHwBinder*&gt;(out-&gt;get()), *flat, in);
        }
    }
    return BAD_TYPE;
}

inline static status_t finish_unflatten_binder(
    BpHwBinder* /*proxy*/, const flat_binder_object&amp; /*flat*/,
    const Parcel&amp; /*in*/)
{
    return NO_ERROR;
}

```

# flat_binder_object

在/bionic/libc/kernel/uapi/linux/android/binder.h文件中可以找到这个struct。 可以确定这几个对象：
- hdr.type：存储binder的type
- flags：标记，会记录与binder相关的一些状态
- binder: 当type=BINDER_TYPE_BINDER时，它指向Binder对象的弱引用
- handle：当type=BINDER_TYPE_HANDLE时，handle对应着binder驱动中 其他进程的binder
- cookie：当type=BINDER_TYPE_BINDER时，它指向Binder对象

```
#ifdef BINDER_IPC_32BIT
typedef __u32 binder_size_t;
typedef __u32 binder_uintptr_t;
#else
typedef __u64 binder_size_t;
typedef __u64 binder_uintptr_t;
#endif
struct binder_object_header {<!-- -->
  __u32 type;
};
struct flat_binder_object {<!-- -->
  struct binder_object_header hdr;
  __u32 flags;
  union {<!-- -->
    binder_uintptr_t binder;
    __u32 handle;
  };
  binder_uintptr_t cookie;
};

```

### ProcessState.getStrongProxyForHandle

大致流程如下：
1. 通过lookupHandleLocked传入handle，获取到handle_entry对象，如果对象不存在就创建一个空的。1. 如果handle_entry中的binder对象为空，那么会根据handle创建一个BpBinder放到里面，并且将这个BpBinder返回。1. 如果handle_entry中的binder对象存在，那么就直接返回。
那么，BpBinder.create(handle)的逻辑是什么？

```
sp&lt;IBinder&gt; ProcessState::getStrongProxyForHandle(int32_t handle)
{
    sp&lt;IBinder&gt; result;

    AutoMutex _l(mLock);

    handle_entry* e = lookupHandleLocked(handle);

    if (e != NULL) {
        // We need to create a new BpBinder if there isn't currently one, OR we
        // are unable to acquire a weak reference on this current one.  See comment
        // in getWeakProxyForHandle() for more info about this.
        IBinder* b = e-&gt;binder;
        if (b == NULL || !e-&gt;refs-&gt;attemptIncWeak(this)) {
            if (handle == 0) {
                Parcel data;
                status_t status = IPCThreadState::self()-&gt;transact(
                        0, IBinder::PING_TRANSACTION, data, NULL, 0);
                if (status == DEAD_OBJECT)
                   return NULL;
            }

            b = BpBinder::create(handle);
            e-&gt;binder = b;
            if (b) e-&gt;refs = b-&gt;getWeakRefs();
            result = b;
        } else {
            // This little bit of nastyness is to allow us to add a primary
            // reference to the remote proxy when this team doesn't have one
            // but another team is sending the handle to us.
            result.force_set(b);
            e-&gt;refs-&gt;decWeak(this);
        }
    }

    return result;
}

ProcessState::handle_entry* ProcessState::lookupHandleLocked(int32_t handle)
{
    const size_t N=mHandleToObject.size();
    if (N &lt;= (size_t)handle) {
        handle_entry e;
        e.binder = NULL;
        e.refs = NULL;
        status_t err = mHandleToObject.insertAt(e, N, handle+1-N);
        if (err &lt; NO_ERROR) return NULL;
    }
    return &amp;mHandleToObject.editItemAt(handle);
}

```

### BpBinder.create()

根据源码可知，BpBinder其实就是一个封装了handle与调用方uid的一个对象。 也可知，其实在进程间传递的binder对象中的数据，主要就是handle与调用方的uid。

```
BpBinder* BpBinder::create(int32_t handle) {
    int32_t trackedUid = -1;
    if (sCountByUidEnabled) {
        trackedUid = IPCThreadState::self()-&gt;getCallingUid();
        ————————————————————————省略
    }
    return new BpBinder(handle, trackedUid);
}

```

# Parcel读写binder总结

走完整个流程的源码解析，再看下流程图就会清晰很多了。 笔者认为有以下要点：
- native层的binder信息会以flat_binder_object的结构存储在Parcel中。
- flat_binder_object.hdr.type为BINDER_TYPE_BINDER代表是本地的binder，那么flat_binder_object中会存储真实的binder。
- flat_binder_object.hdr.type为BINDER_TYPE_BINDER代表远端的binder，那么flat_binder_object中会存储调用远端binder的handle。
- 当binder是远端的时，本地进程实际存储的是BpBinder对象，其中封装了调用远端binder的handle与本地的uid。

<img src="https://img-blog.csdnimg.cn/20201212101436937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> <img src="https://img-blog.csdnimg.cn/20201212101356997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">