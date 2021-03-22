#Handler源码解析
# 前提概要

Hanlder、Looper和MessageQueue算是android中的一大要点，关于其的解说也数不胜数，但他人的终究是他人的。 笔者自己从源码的角度对其深入了解一番，记录成此篇文章。

# 概述

**Looper**：每个线程只有一个Looper，它负责管理MessageQueue，会不断的从MessageQueue中取出消息，并将消息分发给Handler处理。主线程在初始化的时候会创建一个Looper。 **MessageQueue**：用来存放Message的队列，由Looper负责管理。 **Handler**：它能把消息发送给Looper管理的MessageQueue，并负责处理Looper分给它的消息。

# Handler源码

## 构造函数

```
    public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class&lt;? extends Handler&gt; klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &amp;&amp;
                    (klass.getModifiers() &amp; Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

```

主要操作如下：
1. 获取到当前的looper。1. 获取到当前的MessageQueue1. mCallback 是一个接口，其中包括一个回调方法，可以进行message的拦截过滤，但是一般情况不用，为null。(后面会再提到)1. mAsynchronous 表示执行的过程是异步还是同步的。一般情况默认异步。
## 发送Message到MessageQueue

Handler发送Message的方法有多种，比较常见的有post(),sendMessage(),sendMessageDelayed(),sendEmptyMessage()等等，但是他们最终其实都会到enqueueMessage()这个方法，源码如下：

```
   public final boolean sendEmptyMessage(int what)
    {
        return sendEmptyMessageDelayed(what, 0);
    }

    public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageDelayed(msg, delayMillis);
    }

    public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageAtTime(msg, uptimeMillis);
    }

    public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis &lt; 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }

    public final boolean sendMessageAtFrontOfQueue(Message msg) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, 0);
    }

```

```
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

```

msg.target而通过查询Message源码我们也可以看到就是一个Handler，其中把this赋值给它。并且设置是异步还是同步操作。 最终把msg放入了MessageQueue中。

## 处理消息

我们使用Handler时，都是通过定义handleMessage来实现消息处理的，一般如下定义：

```
	   mHandler = new Handler() {
       public void handleMessage(Message msg) {
          // process incoming messages here
       }
   };

```

而当我们在Handler中看其源码时，却发现它是一个空方法：

```
public void handleMessage(Message msg) {
}

```

所以我们其实只需要找到调用它的地方：

```
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

```

此处我们就看到了之前在构造函数中定义的mCallback，我们看到只有mCallback为null或者说mCallback.handleMessage()返回false的时候会执行我们定义的handleMessage()会执行。

当然，相信好奇的同学还是会去查看以下mCallback的源码：

```
    final Callback mCallback;

```

```
    public interface Callback {
        public boolean handleMessage(Message msg);
    }

```

于是我们发现其实它就是一个回调函数，虽然其中的方法handleMessage（）和Handler中的方法同名，但是两者不是一个概念。

# Looper源码

## sThreadLocal变量

```
    // sThreadLocal.get() will return null unless you've called prepare().
    static final ThreadLocal&lt;Looper&gt; sThreadLocal = new ThreadLocal&lt;Looper&gt;();

```

sThreadLocal是一个ThreadLocal类的实例，他负责存储当先线程的Looper实例。

ThreadLocal：每个使用该变量的线程提供独立的变量副本，每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。ThreadLocal内部是通过map进行实现的；

##构造函数

```
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }

```

Looper构造函数非常简单，就是创建一个MessageQueue并且获取到当前的线程。

## Looper.prepare()

```
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

```

此处首先去查看sThreadLocal中是否有Looper实例，如果已经有的话，那么就报异常，这是为了保证一个线程只有一个Looper存在。如果为空的话就创建Looper的实例。非常标准的单例模式。

## Looper.loop()

```
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println("&gt;&gt;&gt;&gt;&gt; Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long traceTag = me.mTraceTag;
            if (traceTag != 0) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            if (logging != null) {
                logging.println("&lt;&lt;&lt;&lt;&lt; Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }

```

代码比较长，但是逻辑比较简单。
1. 获取到当前的Looper，如果为空就报异常，所以调用loop（）之前首先要调用prepare（）创建实例。1. 获取到MessageQueue然后进入死循环遍历，如果queue为空，那么就跳出循环。1. 此处比较关键的代码如下：
```
		try {
                msg.target.dispatchMessage(msg);
            }

```

调用当前Message的Handler的dispatchMessage（）方法。一般情况下也就是调用了我们在初始化Handler时定义的handleMessage（）方法。