#android Binder 实现异步回调
# 场景描述

此时有A、B两个进程，期望A与B实现进程间通信，并且在逻辑执行完毕后再A进程触发回调。期望过程如下：
1. A进程 调用function()1. B进程 触发function()1. B进程 调用callback()1. A进程 触发callback()
# 执行效果

笔者demo中通过打log来记录方法的执行。 执行过程大致如下：
1. 主进程 bindService1. remote进程 执行function()，在function中调用callback()。1. 主进程 执行callback() <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/910.png" alt="在这里插入图片描述"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/911.png" alt="在这里插入图片描述">
# 代码

主要内容如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/912.png" alt="在这里插入图片描述">

### MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

    private IServer iServer;
    private ServiceConnection serviceConnection = new ServiceConnection() {<!-- -->
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {<!-- -->
            iServer = IServer.Stub.asInterface(service);
            System.out.println("onServiceConnected");
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {<!-- -->
            System.out.println("onServiceDisconnected");
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {<!-- -->
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        Button btnBind = findViewById(R.id.btn_bind);
        btnBind.setOnClickListener(new View.OnClickListener() {<!-- -->
            @Override
            public void onClick(View v) {<!-- -->
                //绑定service
                Intent intent = new Intent(MainActivity.this, RemoteTestService.class);
                bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE);
            }
        });

        Button btnTest = findViewById(R.id.btn_test);
        btnTest.setOnClickListener(new View.OnClickListener() {<!-- -->
            @Override
            public void onClick(View v) {<!-- -->
                try {<!-- -->
                    //注册回调方法
                    iServer.setCallBack(new ICallBack.Stub() {<!-- -->
                        @Override
                        public void callback() throws RemoteException {<!-- -->
                            System.out.println("ICallBack callback()");
                        }
                    });
                    //触发远程调用
                    iServer.testFunction();
                } catch (Exception e) {<!-- -->
                    e.printStackTrace();
                }
            }
        });
    }

    @Override
    protected void onDestroy() {<!-- -->
        super.onDestroy();
        unbindService(serviceConnection);
    }
}

```

### RemoteTestService.java

```
/**
 * 进程间通信的service
 */
public class RemoteTestService extends Service {<!-- -->

    private final ServerBinder serverBinder = new ServerBinder();

    @Override
    public IBinder onBind(Intent intent) {<!-- -->
        return serverBinder;
    }

}

```

### ServerBinder.java

```
/**
 * IServer.aidl的实现
 */
public class ServerBinder extends IServer.Stub {<!-- -->

    @Override
    public void testFunction() throws RemoteException {<!-- -->
        System.out.println("IServer testFunction()");

        //调用callback
        int i = RemoteCallbackManager.getInstance().callbackList.beginBroadcast();
        while (i &gt; 0) {<!-- -->
            i--;
            try {<!-- -->
                RemoteCallbackManager.getInstance().callbackList.getBroadcastItem(i).callback();
                RemoteCallbackManager.getInstance().callbackList.unregister(RemoteCallbackManager.getInstance().callbackList.getBroadcastItem(i));
            } catch (RemoteException e) {<!-- -->
                // The RemoteCallbackList will take care of removing
                // the dead object for us.
            }
        }
        RemoteCallbackManager.getInstance().callbackList.finishBroadcast();
    }

    @Override
    public void setCallBack(ICallBack callBack) throws RemoteException {<!-- -->
        //绑定callback
        RemoteCallbackManager.getInstance().callbackList.register(callBack);
    }
}

```

### RemoteCallbackManager.java

```
/**
 * 用于存储RemoteCallbackList的单例
 */
public class RemoteCallbackManager {<!-- -->

    public RemoteCallbackList&lt;ICallBack&gt; callbackList;

    private RemoteCallbackManager() {<!-- -->
        callbackList = new RemoteCallbackList&lt;&gt;();
    }

    private static class Host {<!-- -->
        private static RemoteCallbackManager instance = new RemoteCallbackManager();
    }

    public static RemoteCallbackManager getInstance() {<!-- -->
        return Host.instance;
    }
}

```

### IServer.aidl

```
interface IServer {<!-- -->
   void testFunction();
   void setCallBack(com.example.bindercallbacktest.ICallBack callback);
}

```

### ICallBack.aidl

```
interface ICallBack {<!-- -->
    void callback();
}

```

### AndroidManifest.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.bindercallbacktest"&gt;

    &lt;application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"&gt;
        &lt;activity android:name=".MainActivity"&gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;

        &lt;service
            android:name=".RemoteTestService"
            android:process=":remote" /&gt;

    &lt;/application&gt;

&lt;/manifest&gt;

```