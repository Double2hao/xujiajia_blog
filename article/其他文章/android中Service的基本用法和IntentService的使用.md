#android中Service的基本用法和IntentService的使用
> 
  


> 
 至于Service是什么此，相信大家都有一定的了解，篇也不多做概述了。 
 此篇主要是讲一下Service的基本用法和IntentService的使用。 
   
 首先是说一下Service中主要的四个方法： 
   
 onCreate():在服务创建的时候调用。 
 onStartCommand():在每次服务启动的时候调用。 
 此处可能很多读者会问
 那么onStartCommand()与onCreate()有什么区别呢
 ？onCreate()方法是在服务第一期创建的时候调用，而onStartCommand()则在每次启动服务的时候都会调用。如果在startService(intent)之后继续还继续执行startService(intent)这个方法，那么只会有onStartCommand()可以执行。 
 onDestroy():在服务销毁的时候调用。 
 onBind():此方法在服务被绑定时调用。此方法是让Service与某个Activity绑定，使该Activity可以与该Service更密切。如下面的实例中，就实现了后台开始下载和查看下载进度的方法。另外，如果仅仅只进行了Service与Activity的绑定，那么onStartCommand()中的方法是不会执行的，其实也比较好理解，因为我们已经把需要实现的内容写在ServiceConnection()中了。（此处一下子理解不了也没事，看一下代码分分钟就弄懂了） 
 <blockquote> 
    
 

另外是IntentService。

试想如果我们要实现一个从网上下载图片，然后更新到Activity中的后台程序。那么我们就需要在Service中用子线程实现功能，并且在功能实现之后关闭子线程。如果直接在Service中实现该功能的话，很容易出现ANR的情况（Appication Not Responding）。此方法虽然不复杂，但是总会有一些程序员忘记开启线程或者忘记关闭服务。

于是Android就专门提供了IntentService类，这个类很好地解决了前面的问题。

 

接下来说一下IntentService中的主要方法：

onHandleIntent():这个方法中就是实现主要功能了。使用此方法不用担心ANR的问题，因为这个方法已经是在子线程中运行的了。

onDestroy():根据IntentService的特性，这个服务在运行结束之后是会自动停止的，此处可以打印日志来确定服务已经停止了。

 

好了说了这么多久直接看一下测试代码吧，先上图：

>  
   


 

<img alt="" class="has" height="700" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2250.png" width="400">

 

>  
 <blockquote> 
  <blockquote> 
     
  

>  
 <blockquote> 
    
 

>  
 <blockquote> 
    
 

>  
 <blockquote> 
    
 

>  
   


>  
   


>  
 <blockquote> 
    
 

>  
   


>  
   


> 
  


> 
  


> 
  


> 
  


> 
  


> 
  


> 
  


>  
   


> 
  


> 
  


> 
  


> 
  


> 
  


> 
  


 

 

 

 

MainActivity:

 

```
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (MyService.DownloadBinder) service;
            //开始下载
            downloadBinder.startDownload();
            //查看下载进度
            downloadBinder.getProgress();
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        startService = (Button) findViewById(R.id.start_service);
        stopService = (Button) findViewById(R.id.stop_service);

        bindService = (Button) findViewById(R.id.bind_service);
        unbindService = (Button) findViewById(R.id.unbind_service);

        startIntentService = (Button) findViewById(R.id.start_intent_service);

        startService.setOnClickListener(this);
        stopService.setOnClickListener(this);
        bindService.setOnClickListener(this);
        unbindService.setOnClickListener(this);
        startIntentService.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.start_service:
                Intent startIntent = new Intent(this, MyService.class);
                startService(startIntent);
                break;
            case R.id.stop_service:
                Intent stopIntent = new Intent(this, MyService.class);
                stopService(stopIntent);
                break;
            case R.id.bind_service:
                Intent bindIntent = new Intent(this, MyService.class);
                //在调用bindService时onStartCommand的内容将不会执行。
                //其实也比较好理解，因为我们已经将需要执行的东西写到了ServiceConnection实例中。
                bindService(bindIntent, connection, BIND_AUTO_CREATE);
                break;
            case R.id.unbind_service:
                unbindService(connection);
                break;
            case R.id.start_intent_service:
                Log.d("MainActivity", "Thread id is " + Thread.currentThread().getId());
                Intent intentService = new Intent(this, MyIntentService.class);
                startService(intentService);
                break;
            default:
                break;
        }
    }

}
```

 

 

 

 

 

MyService:

 

```
import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;
import android.util.Log;

public class MyService extends Service {

	private DownloadBinder mBinder = new DownloadBinder();

	//实现在bind_service中实现的功能
	//此处主要是开始下载和查看下载进度的方法
	//此处为了便于理解也仅仅是打印日志，并没有实现真正的功能
	class DownloadBinder extends Binder {

		public void startDownload() {
			new Thread(new Runnable() {
				@Override
				public void run() {
					// start downloading
				}
			}).start();
			Log.d("MyService", "startDownload executed");
		}

		public int getProgress() {
			Log.d("MyService", "getProgress executed");
			return 0;
		}

	}

	//DownloadBinder需要在onBind方法中返回实例
	//绑定 service的功能才会实现
	@Override
	public IBinder onBind(Intent intent) {
		Log.d("MyService", "onBind executed");
		return mBinder;
	}

	@Override
	public void onCreate() {
		super.onCreate();
		Log.d("MyService", "onCreate executed");
	}

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		Log.d("MyService", "onStartCommand executed");
		new Thread(new Runnable() {  
	        @Override  
	        public void run() {  
	            // do something here
	        }  
	    }).start();
		return super.onStartCommand(intent, flags, startId);
	}

	@Override
	public void onDestroy() {
		super.onDestroy();
		Log.d("MyService", "onDestroy executed");
	}

}
```

 

 

 

 

 

MyIntentService:

 

```
import android.app.IntentService;
import android.content.Intent;
import android.util.Log;

public class MyIntentService extends IntentService {

	public MyIntentService() {
		super("MyIntentService");
	}

	//这个方法中可以去处理一些具体的逻辑，而且不用担心ANR的问题
	//因为这个方法已经是在子线程中运行的了
	@Override
	protected void onHandleIntent(Intent intent) {
		Log.d("MyIntentService", "Thread id is " + Thread.currentThread().getId());
	}

	@Override
	public void onDestroy() {
		super.onDestroy();
		Log.d("MyIntentService", "onDestroy executed");
	}

}

```

 

 

 

 

 

activity_main:

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" &gt;

    &lt;Button
        android:id="@+id/start_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Start Service" /&gt;

    &lt;Button
        android:id="@+id/stop_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Stop Service" /&gt;

    &lt;Button
        android:id="@+id/bind_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Bind Service" /&gt;

    &lt;Button
        android:id="@+id/unbind_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Unbind Service" /&gt;

    &lt;Button
        android:id="@+id/start_intent_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Start IntentService" /&gt;

&lt;/LinearLayout&gt;
```

 

 

 

 

 

千万不能忘了在此处对Service进行注册。

AndroidManifest:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.servicetest"
    android:versionCode="1"
    android:versionName="1.0" &gt;

    &lt;uses-sdk
        android:minSdkVersion="14"
        android:targetSdkVersion="17" /&gt;

    &lt;application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" &gt;
        &lt;activity
            android:name="com.example.servicetest.MainActivity"
            android:label="@string/app_name" &gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;
                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;

        &lt;service android:name=".MyService" &gt;
        &lt;/service&gt;
        &lt;service android:name=".MyIntentService"&gt;&lt;/service&gt;
    &lt;/application&gt;

&lt;/manifest&gt;
```

 

 

 

 

 

 