#由浅入深 学习 Android Binder（二）- bindService流程
>  
 Android Binder系列文章：           


# 概述

前文已经讲过了binder在aidl的IPC中所起到的作用。其中的Binder是通过ServiceConnection+bindService获取到的。

>  
 前文链接： 


本文承接前文，详细探索一下从bindService到ServiceConnection.onServiceConnected()的流程。

# bindService流程

bindService的流程其实有两种场景：
1. Service冷启动1. Service热启动
本文为了分析的更加全面，因此主要分析”Service冷启动“的场景。

笔者此处先列出流程，读者可以稍微心里有数。 这也算是后文具体分析的一个目录，更方便于有一个整体的理解。

>  
 有兴趣的读者可以通过打断点，输出任务栈等方式验证。 


在“Service冷启动”的场景下，bindService到onServiceConnected()的整个流程如下。 （流程图在文章结尾）
- ContextWrapper.bindService()- ContextImpl.bindService()- ContextImpl. bindServiceCommon()- ActivityManagerService.bindService()- ActiveServices.bindServiceLocked()- ActiveServices.requestServiceBindingLocked()- ActivityThread.ApplicationThread.scheduleBindService()- ActivityThread.sendMessage()- ActivityThread.handleBindService()- ActivityManagerService.publishService()- ActiveService.publishServiceLocked()- LoadedApk.ServiceDispatcher.InnerConnection.connected()- LoadedApk.ServiceDispatcher.connected()- ActivityThread.post()- LoadedApk.ServiceDispatcher. - RunConnection.run()- LoadedApk.ServiceDispatcher.doConnected()- ServiceConnection.onServiceConnected()
>  
 对热启动有兴趣的读者也可以自行看下源码。 在“Service热启动”的场景下，整个流程如下: 
 - ContextWrapper.bindService()- ContextImpl.bindService()- ContextImpl.bindServiceCommon()- ActivityManagerService.bindService()- ActiveServices.bindServiceLocked()- LoadedApk.ServiceDispatcher.InnerConnection.connected()- LoadedApk.ServiceDispatcher.connected()- ActivityThread.post()- LoadedApk.ServiceDispatcher. RunConnection.run()- LoadedApk.ServiceDispatcher.doConnected()- ServiceConnection.onServiceConnected() 


### ContextWrapper.bindService()

```
    Context mBase;

    @Override
    public boolean bindService(Intent service, ServiceConnection conn,
            int flags) {<!-- -->
        return mBase.bindService(service, conn, flags);
    }

```

四大组件调用bindService()的时候，实际上调用的是他们父类ContextWrapper的bindService()。 ContextWrapper中调用的mBase实际是ContextImpl。

### ContextImpl.bindService()

```
    @Override
    public boolean bindService(Intent service, ServiceConnection conn,
            int flags) {<!-- -->
        warnIfCallingFromSystemProcess();
        return bindServiceCommon(service, conn, flags, mMainThread.getHandler(), getUser());
    }


```

直接看bindServiceCommon()代码。

### ContextImpl.bindServiceCommon()

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

后续执行的主要代码其实就是下面这行.

```
            int res = ActivityManager.getService().bindService(
                mMainThread.getApplicationThread(), getActivityToken(), service,
                service.resolveTypeIfNeeded(getContentResolver()),
                sd, flags, getOpPackageName(), user.getIdentifier());

```

我们直接看下ActivityManager的代码，看看getService是什么。

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

这里可以看到，getService最终是获取到了IActivityManager的Binder对象然后返回。

搜一下源码就可以知道，frameworks的代码只有ActivityManagerService继承了IActivityManager.Stub。

因此，此处会进入到ActivityManagerService的bindService()方法。

### ActivityManagerService.bindService()

```
    public int bindService(IApplicationThread caller, IBinder token, Intent service,
            String resolvedType, IServiceConnection connection, int flags, String callingPackage,
            int userId) throws TransactionTooLargeException {<!-- -->
        enforceNotIsolatedCaller("bindService");

        // Refuse possible leaked file descriptors
        if (service != null &amp;&amp; service.hasFileDescriptors() == true) {<!-- -->
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }

        if (callingPackage == null) {<!-- -->
            throw new IllegalArgumentException("callingPackage cannot be null");
        }

        synchronized(this) {<!-- -->
            return mServices.bindServiceLocked(caller, token, service,
                    resolvedType, connection, flags, callingPackage, userId);
        }
    }

```

最终执行到的逻辑其实是下面的代码：

```
        synchronized(this) {<!-- -->
            return mServices.bindServiceLocked(caller, token, service,
                    resolvedType, connection, flags, callingPackage, userId);
        }

```

```
    final ActiveServices mServices;

```

继续看下ActiveServices这个类的bindServiceLocked方法。

```

  int bindServiceLocked(IApplicationThread caller, IBinder token, Intent service,
      String resolvedType, final IServiceConnection connection, int flags,
      String callingPackage, final int userId) throws TransactionTooLargeException {<!-- -->

——————————————省略

      ConnectionRecord c = new ConnectionRecord(b, activity,
          connection, flags, clientLabel, clientIntent);

      if (s.app != null &amp;&amp; b.intent.received) {<!-- -->
        // Service is already running, so we can immediately
        // publish the connection.
        try {<!-- -->
          c.conn.connected(s.name, b.intent.binder, false);
        } catch (Exception e) {<!-- -->
          Slog.w(TAG, "Failure sending service " + s.shortName
              + " to connection " + c.conn.asBinder()
              + " (in " + c.binding.client.processName + ")", e);
        }

        // If this is the first app connected back to this binding,
        // and the service had previously asked to be told when
        // rebound, then do so.
        if (b.intent.apps.size() == 1 &amp;&amp; b.intent.doRebind) {<!-- -->
          requestServiceBindingLocked(s, b.intent, callerFg, true);
        }
      } else if (!b.intent.requested) {<!-- -->
        requestServiceBindingLocked(s, b.intent, callerFg, false);
      }
      
——————————————省略

}


```

```
final class ConnectionRecord {<!-- -->
——————————————省略
    final IServiceConnection conn;  // The client connection.
——————————————省略
}

```

如上可以看到，逻辑如下：
- 如果service已经启动，则会调用ConnectionRecord.conn.connect();- 如果service没有启动，则会调用requestServiceBindingLocked()；
由于我们是假设的”Service冷启动“场景，因此我们直接看下requestServiceBindingLocked()的逻辑。

### ActiveServices.requestServiceBindingLocked()

```
    private final boolean requestServiceBindingLocked(ServiceRecord r, IntentBindRecord i,
            boolean execInFg, boolean rebind) throws TransactionTooLargeException {<!-- -->
        if (r.app == null || r.app.thread == null) {<!-- -->
            // If service is not currently running, can't yet bind.
            return false;
        }
        if (DEBUG_SERVICE) Slog.d(TAG_SERVICE, "requestBind " + i + ": requested=" + i.requested
                + " rebind=" + rebind);
        if ((!i.requested || rebind) &amp;&amp; i.apps.size() &gt; 0) {<!-- -->
            try {<!-- -->
                bumpServiceExecutingLocked(r, execInFg, "bind");
                r.app.forceProcessStateUpTo(ActivityManager.PROCESS_STATE_SERVICE);
                r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind,
                        r.app.repProcState);
                if (!rebind) {<!-- -->
                    i.requested = true;
                }
                i.hasBound = true;
                i.doRebind = false;
            } catch (TransactionTooLargeException e) {<!-- -->
                // Keep the executeNesting count accurate.
                if (DEBUG_SERVICE) Slog.v(TAG_SERVICE, "Crashed while binding " + r, e);
                final boolean inDestroying = mDestroyingServices.contains(r);
                serviceDoneExecutingLocked(r, inDestroying, inDestroying);
                throw e;
            } catch (RemoteException e) {<!-- -->
                if (DEBUG_SERVICE) Slog.v(TAG_SERVICE, "Crashed while binding " + r);
                // Keep the executeNesting count accurate.
                final boolean inDestroying = mDestroyingServices.contains(r);
                serviceDoneExecutingLocked(r, inDestroying, inDestroying);
                return false;
            }
        }
        return true;
    }

```

如上，如果service还没有bind，就会触发bind逻辑，最终会走到这行代码：

```
r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind,
                        r.app.repProcState);

```

r是ServiceRecord app是ProcessRecord thread是IApplicationThread 所以最终是IApplicationThread调用了scheduleBindService。

我们全局搜索一下是哪个类实现了IApplicationThread.Stub，可以找到ActivityThread.ApplicationThread。 于是下面就分析ActivityThread.ApplicationThread.scheduleBindService()

### ActivityThread.ApplicationThread.scheduleBindService()

```
        public final void scheduleBindService(IBinder token, Intent intent,
                boolean rebind, int processState) {<!-- -->
            updateProcessState(processState, false);
            BindServiceData s = new BindServiceData();
            s.token = token;
            s.intent = intent;
            s.rebind = rebind;

            if (DEBUG_SERVICE)
                Slog.v(TAG, "scheduleBindService token=" + token + " intent=" + intent + " uid="
                        + Binder.getCallingUid() + " pid=" + Binder.getCallingPid());
            sendMessage(H.BIND_SERVICE, s);
        }

```

最终会sendMessage,id是H.BIND_SERVICE。 于是，到handleMessage中查看实现。

### ActivityThread.H.handleMessage()

```
public void handleMessage(Message msg) {<!-- -->
————————————————省略

case BIND_SERVICE:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "serviceBind");
                    handleBindService((BindServiceData)msg.obj);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
————————————————省略
}


```

最终执行到了ActivityThread.handleBindService()中。

### ActivityThread.handleBindService()

```
    private void handleBindService(BindServiceData data) {<!-- -->
————————————————省略
       if (!data.rebind) {<!-- -->
                        IBinder binder = s.onBind(data.intent);
                        ActivityManager.getService().publishService(
                                data.token, data.intent, binder);
                    } else {<!-- -->
                        s.onRebind(data.intent);
                        ActivityManager.getService().serviceDoneExecuting(
                                data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
                    }
————————————————省略
    }

```

由于我们是冷启动，所以rebind为false，因此最终会走到publishService的逻辑中。

ActivityManager.getService()前面已经分析过，这是个IActivityManager对象。 是由ActivityManagerService来实现的IActivityManager.Stub()。

### ActivityManagerService.publishService()

```
    final ActiveServices mServices;

    public void publishService(IBinder token, Intent intent, IBinder service) {<!-- -->
————————————————省略

        synchronized(this) {<!-- -->
            if (!(token instanceof ServiceRecord)) {<!-- -->
                throw new IllegalArgumentException("Invalid service token");
            }
            mServices.publishServiceLocked((ServiceRecord)token, intent, service);
        }
    }

```

接着进入到ActiveServices.publishServiceLocked()

### ActiveService.publishServiceLocked()

```
    void     void publishServiceLocked(ServiceRecord r, Intent intent, IBinder service) {<!-- -->
    ————————————————省略
    ConnectionRecord c = clist.get(i);
    ————————————————省略
    c.conn.connected(r.name, service, false);
    ————————————————省略
}

```

```
final class ConnectionRecord {<!-- -->
——————————————省略
    final IServiceConnection conn;  // The client connection.
——————————————省略
}

```

此处最终执行到的是ConnectionRecord.conn.connected，根据上面源码，可以知道执行的是IServiceConnection的connect方法。

此处需要确定下IServiceConnection的具体是哪个类。 全局搜一下，可以发现只有LoadedApk.ServiceDispatcher.InnerConnection 实现了IServiceConnection.Stub，因此此处就是它。

于是，接下来看LoadedApk.ServiceDispatcher.InnerConnection的connect()方法。

### LoadedApk.ServiceDispatcher.InnerConnection.connect()

```
        private static class InnerConnection extends IServiceConnection.Stub {<!-- -->
            final WeakReference&lt;LoadedApk.ServiceDispatcher&gt; mDispatcher;

            InnerConnection(LoadedApk.ServiceDispatcher sd) {<!-- -->
                mDispatcher = new WeakReference&lt;LoadedApk.ServiceDispatcher&gt;(sd);
            }

            public void connected(ComponentName name, IBinder service, boolean dead)
                    throws RemoteException {<!-- -->
                LoadedApk.ServiceDispatcher sd = mDispatcher.get();
                if (sd != null) {<!-- -->
                    sd.connected(name, service, dead);
                }
            }
        }

```

由代码可知，最终执行到 LoadedApk.ServiceDispatcher.connected（）。

```
    static final class ServiceDispatcher {<!-- -->
    ——————————————省略
    
    public void connected(ComponentName name, IBinder service, boolean dead) {<!-- -->
            if (mActivityThread != null) {<!-- -->
                mActivityThread.post(new RunConnection(name, service, 0, dead));
            } else {<!-- -->
                doConnected(name, service, dead);
            }
        }
        
        private final class RunConnection implements Runnable {<!-- -->
            RunConnection(ComponentName name, IBinder service, int command, boolean dead) {<!-- -->
                mName = name;
                mService = service;
                mCommand = command;
                mDead = dead;
            }

            public void run() {<!-- -->
                if (mCommand == 0) {<!-- -->
                    doConnected(mName, mService, mDead);
                } else if (mCommand == 1) {<!-- -->
                    doDeath(mName, mService);
                }
            }

            final ComponentName mName;
            final IBinder mService;
            final int mCommand;
            final boolean mDead;
        }
    ——————————————省略
    }

```

由代码可知，ServiceDispatcher.connected()的逻辑有两种场景：
1. 如果mActivityThread不为空，则通过handler来执行doConnected().1. 如果mActivityThread不为空，则直接执行doConnected().
再看下doConnected的逻辑：

```

        private final ServiceConnection mConnection;
       private final ArrayMap&lt;ComponentName, ServiceDispatcher.ConnectionInfo&gt; mActiveConnections
            = new ArrayMap&lt;ComponentName, ServiceDispatcher.ConnectionInfo&gt;();
        
 ————————————————————省略
        
       public void doConnected(ComponentName name, IBinder service, boolean dead) {<!-- -->
            ServiceDispatcher.ConnectionInfo old;
            ServiceDispatcher.ConnectionInfo info;

            synchronized (this) {<!-- -->
                if (mForgotten) {<!-- -->
                    // We unbound before receiving the connection; ignore
                    // any connection received.
                    return;
                }
                old = mActiveConnections.get(name);
                if (old != null &amp;&amp; old.binder == service) {<!-- -->
                    // Huh, already have this one.  Oh well!
                    return;
                }

                if (service != null) {<!-- -->
                    // A new service is being connected... set it all up.
                    info = new ConnectionInfo();
                    info.binder = service;
                    info.deathMonitor = new DeathMonitor(name, service);
                    try {<!-- -->
                        service.linkToDeath(info.deathMonitor, 0);
                        mActiveConnections.put(name, info);
                    } catch (RemoteException e) {<!-- -->
                        // This service was dead before we got it...  just
                        // don't do anything with it.
                        mActiveConnections.remove(name);
                        return;
                    }

                } else {<!-- -->
                    // The named service is being disconnected... clean up.
                    mActiveConnections.remove(name);
                }

                if (old != null) {<!-- -->
                    old.binder.unlinkToDeath(old.deathMonitor, 0);
                }
            }

            // If there was an old service, it is now disconnected.
            if (old != null) {<!-- -->
                mConnection.onServiceDisconnected(name);
            }
            if (dead) {<!-- -->
                mConnection.onBindingDied(name);
            }
            // If there is a new viable service, it is now connected.
            if (service != null) {<!-- -->
                mConnection.onServiceConnected(name, service);
            } else {<!-- -->
                // The binding machinery worked, but the remote returned null from onBind().
                mConnection.onNullBinding(name);
            }
        }

```

此处逻辑大概如下：
1. 先判断service是否已经在缓存中并且没有断开，如果是，那么代表已经连接，就不需要触发onServiceDisconnected()，直接return。1. 如果不在缓存中，那么就需要重新加入缓存，代表已经连接，然后触发onServiceDisconnected()。1. 还有一种特殊的情况是”在缓存中，但是已经断开“，那么也需要重新加入缓存，然后触发onServiceDisconnected()。
至此，整个流程就已经走完了。

# 总结

大概流程：
1. Client进程调用bindService通知AMS进程。1. AMS进程通知Server进程启动service。1. server进程启动service，并且在启动完成后通知AMS进程。1. AMS接收到server进程bindService的结果，AMS进程通知client进程。1. client进程接收binService结果，client进程调用onServiceConnected()。
"冷启动"场景的bindService流程图如下： <img src="https://img-blog.csdnimg.cn/20201121202423379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70#pic_center" alt="在这里插入图片描述">
