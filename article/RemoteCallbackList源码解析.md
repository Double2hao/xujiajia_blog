#RemoteCallbackList源码解析
# 概要

之前已经写过了RemoteCallbackList的使用，此文就不讲demo了。 对RemoteCallbackList的使用有兴趣的读者可以看笔者之前的文章： 

# 官方描述

>  
 Takes care of the grunt work of maintaining a list of remote interfaces, typically for the use of performing callbacks from a android.app.Service to its clients. In particular, this: 
 - Keeps track of a set of registered IInterface callbacks, taking care to identify them through their underlying unique IBinder (by calling IInterface#asBinder.- Attaches a IBinder.DeathRecipient to each registered interface, so that it can be cleaned out of the list if its process goes away.- Performs locking of the underlying list of interfaces to deal with multithreaded incoming calls, and a thread-safe way to iterate over a snapshot of the list without holding its lock. 
 To use this class, simply create a single instance along with your service, and call its register and unregister methods as client register and unregister with your service. To call back on to the registered clients, use beginBroadcast, getBroadcastItem, and finishBroadcast. If a registered callback’s process goes away, this class will take care of automatically removing it from the list. If you want to do additional work in this situation, you can create a subclass that implements the onCallbackDied method. 


### 笔者翻译大致如下

用来承担保存远程接口的责任，主要用法是为Service的client实现callback，注意点如下：
1. 存储接口callbakc的引用，通过Ibinder来区分它们。（简单说就是，用的Map，key为IBinder）1. 为每个接口设置DeathRecipient，这样在进程销毁的时候，接口可以从存储列表中移除。1. 为每个callback的调用上锁，并且也使用了”快照“的方式。（简单来说就是线程安全）
推荐在单例中使用这个类。 如果需要在进程销毁的时候需要有处理逻辑，可以写在onCallbackDied这个方法中。

# 源码分析

### 存储结构

结构使用的是ArrayMap。

>  
 为什么不用HashMap？ 个人理解，如果存储的数量不是很多的话，HashMap是无法体现出优势的。反而HashMap本身的扩容机制会造成不必要的浪费。 而RemoteCallbackList既然是用于存储进程间的callback，单个进程的callback自然是很少会有大数量的情况的。 


```
    /*package*/ ArrayMap&lt;IBinder, Callback&gt; mCallbacks
            = new ArrayMap&lt;IBinder, Callback&gt;();

```

### callback类

继承自IBinder.DeathRecipient。 这样在进程销毁的时候可以拿到通知，callback可以通过重写onCallbackDied来实现进程销毁后的逻辑。

```
    private final class Callback implements IBinder.DeathRecipient {<!-- -->
        final E mCallback;
        final Object mCookie;

        Callback(E callback, Object cookie) {<!-- -->
            mCallback = callback;
            mCookie = cookie;
        }

        public void binderDied() {<!-- -->
            synchronized (mCallbacks) {<!-- -->
                mCallbacks.remove(mCallback.asBinder());
            }
            onCallbackDied(mCallback, mCookie);
        }
    }

```

### register()
1. 通过对mCallbacks上锁来保证线程的安全。1. 通过linkToDeath，保证在进程销毁的时候，callback可以被通知到。1. 将callback存储到mCallbacks中。
```
    public boolean register(E callback, Object cookie) {<!-- -->
        synchronized (mCallbacks) {<!-- -->
            if (mKilled) {<!-- -->
                return false;
            }
            // Flag unusual case that could be caused by a leak. b/36778087
            logExcessiveCallbacks();
            IBinder binder = callback.asBinder();
            try {<!-- -->
                Callback cb = new Callback(callback, cookie);
                binder.linkToDeath(cb, 0);
                mCallbacks.put(binder, cb);
                return true;
            } catch (RemoteException e) {<!-- -->
                return false;
            }
        }
    }

```

### unregister()
1. 通过对mCallbacks上锁来保证线程的安全。1. 把mCallbacks中移除callback1. 执行unlinkToDeath，避免在进程销毁的时候通知callback
```
    public boolean unregister(E callback) {<!-- -->
        synchronized (mCallbacks) {<!-- -->
            Callback cb = mCallbacks.remove(callback.asBinder());
            if (cb != null) {<!-- -->
                cb.mCallback.asBinder().unlinkToDeath(cb, 0);
                return true;
            }
            return false;
        }
    }

```

### beginBroadcast()
1. 通过对mCallbacks上锁来保证线程的安全。1. 对mCallbacks作快照操作，存储到mActiveBroadcast中。
>  
 为什么有了锁的逻辑，还要用快照来保证线程安全？ 一般的操作流程是Callback.callback-&gt;RemoteCallbackList.unregister。 如果同时有多个线程在执行此流程，那么有可能一个线程执行unregister的操作可能会影响另一个线程的callback。在执行逻辑之前先通过beginBroadcast作快照，能够避免多线程之间的互相影响。 


```
    public int beginBroadcast() {<!-- -->
        synchronized (mCallbacks) {<!-- -->
            if (mBroadcastCount &gt; 0) {<!-- -->
                throw new IllegalStateException(
                        "beginBroadcast() called while already in a broadcast");
            }
            
            final int N = mBroadcastCount = mCallbacks.size();
            if (N &lt;= 0) {<!-- -->
                return 0;
            }
            Object[] active = mActiveBroadcast;
            if (active == null || active.length &lt; N) {<!-- -->
                mActiveBroadcast = active = new Object[N];
            }
            for (int i=0; i&lt;N; i++) {<!-- -->
                active[i] = mCallbacks.valueAt(i);
            }
            return N;
        }
    }

```

### finishBroadcast()

在callback执行结束之后调用，主要用于注销callback事件。

```
    public void finishBroadcast() {<!-- -->
        synchronized (mCallbacks) {<!-- -->
            if (mBroadcastCount &lt; 0) {<!-- -->
                throw new IllegalStateException(
                        "finishBroadcast() called outside of a broadcast");
            }

            Object[] active = mActiveBroadcast;
            if (active != null) {<!-- -->
                final int N = mBroadcastCount;
                for (int i=0; i&lt;N; i++) {<!-- -->
                    active[i] = null;
                }
            }

            mBroadcastCount = -1;
        }
    }

```