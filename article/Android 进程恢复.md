#Android 进程恢复
# 概述

在android多进程的场景中，进程被意外杀死的情况非常场景，可能是用户手动杀死进程，也有可能是操作系统内存不够主动回收。 某个进程被杀死，会导致其他进程与这个进程的通信失效，因此，一旦发生这种情况，就需要在其他进程，对这个被杀死的进程作“进程恢复”。

# 进程恢复重要步骤
1. 在进程死亡的时候重启进程。1. 检查进程间通信逻辑，如果失效需要做恢复。 此处的恢复，简单来说就是一些初始化操作，可能包括io，网络等操作。
# Demo

>  
 Demo 的github地址：  

1. 在MainActivity中启动WebViewActivity1. WebViewActivity中通过IPC去主进程获取需要跳转的url。1. 杀掉MainActivity，同时也是杀掉主进程。1. 这时候WebViewActivity再次尝试去主进程获取需要跳转的url。 由于主进程已经被杀死，因此IPC会失效，一旦发现IPC失效，那么就尝试恢复服务，然后再继续IPC的逻辑。1. 最终可以实现，即使主进程被杀死，webview进程与主进程的通信仍能继续。
### 重启进程逻辑

在主进程IPC的Service与webview进程建立成功时，给该binder注册死亡事件，即重启service。 代码如下：

```
  private ServiceConnection serviceConnection = new ServiceConnection() {<!-- -->
    @Override public void onServiceConnected(ComponentName name, IBinder service) {<!-- -->
      try {<!-- -->
        webViewServer = IWebViewServer.Stub.asInterface(service);
        webViewServer.asBinder().linkToDeath(new IBinder.DeathRecipient() {<!-- -->
          @Override public void binderDied() {<!-- -->
            startWebViewServerService();//如果断开就重启
          }
        }, 0);
      } catch (RemoteException e) {<!-- -->
        e.printStackTrace();
      }
    }

    @Override public void onServiceDisconnected(ComponentName name) {<!-- -->

    }
  };

```

### ipc方法失效后，作恢复逻辑

每次调用ipc方法的时候，都去check一下是否需要作恢复逻辑。

```
public class WebViewServerBinder extends IWebViewServer.Stub {<!-- -->
  @Override public String getUrl() throws RemoteException {<!-- -->
    checkRecovery();
    return TestDataManager.getInstance().getUrl();
  }

  /*
  确认是否需要执行恢复逻辑
   */
  public void checkRecovery() {<!-- -->
    if (TextUtils.isEmpty(TestDataManager.getInstance().getUrl())) {<!-- -->
      TestDataManager.getInstance().initUrlData();
    }
  }
}

```

# github

对具体实现以及细节感兴趣的读者可以自行看下实现逻辑。

>  
 Demo 的github地址：  
