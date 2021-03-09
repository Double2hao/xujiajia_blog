#android 后台定时提醒（Service,AlarmManager的使用）
> 
 笔者最近将工具书上Service的有关内容都学习了一下，于是打算做一个小应用来练一下手了。
 考虑到自己每次在敲代码或者打游戏的时候总是会不注意时间，一不留神就对着电脑连续3个小时以上，对眼睛的伤害还是挺大的，重度近视了可是会遗传给将来的孩子的呀，可能老婆都跟别人跑了。
 于是，为了保护眼睛，笔者便做了个如下的应用：
 （界面为了便于让新手理解，所以做的比较简单，并且没有设置背景图片，也没有设置APP桌面图片，有心的读者完全可以放上自己或者对象的图片，然后做的比较个人化）
 
 **打开后效果：**
 
 <img src="https://img-blog.csdn.net/20151108212436259?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt="">
 
 
 **时间到之后有后台提醒：**
 
 <img src="https://img-blog.csdn.net/20151108170400618?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt="">
 
 <img src="https://img-blog.csdn.net/20151108170510054?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt="">
 
 
 好了，接下来说一下做这样一个APP主要涉及到的知识点：
 
 **Service:**使用service，便可以在程序即使后台运行的时候，也能够做出相应的提醒，并且不影响手机进行其他工作。
 **AlarmManager:**此知识点主要是用来计时，具体的在代码的注释中写的很详细。
 **notification:**此知识点就是用作通知的显示了，具体的可以参考笔者的另一篇博客：
 
 
 
 MainActivity:
 
 <pre><code class="language-java">import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.Window;
import android.widget.Toast;

public class MainActivity extends Activity {

	private Intent intent;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		//取消标题栏
		requestWindowFeature(Window.FEATURE_NO_TITLE);
		//由于主要是用于测试，并且便于新手理解，所以activity_main布局写的很简单
		setContentView(R.layout.activity_main);
		intent = new Intent(this, LongRunningService.class);
		//开启关闭Service
		startService(intent);

		//设置一个Toast来提醒使用者提醒的功能已经开始
		Toast.makeText(MainActivity.this,"提醒的功能已经开启，关闭界面则会取消提醒。",Toast.LENGTH_LONG).show();
	}

	@Override
	protected void onDestroy() {
		super.onDestroy();
		//在Activity被关闭后，关闭Service
		stopService(intent);
	}
}</code></pre>
 
 
 
 LongRunningService:
 
 <pre><code class="language-java">import android.app.AlarmManager;
import android.app.PendingIntent;
import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.SystemClock;

public class LongRunningService extends Service {


	@Override
	public IBinder onBind(Intent intent) {
		return null;
	}

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {

		AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
		//读者可以修改此处的Minutes从而改变提醒间隔时间
		//此处是设置每隔90分钟启动一次
		//这是90分钟的毫秒数
		int Minutes = 90*60*1000;
		//SystemClock.elapsedRealtime()表示1970年1月1日0点至今所经历的时间
		long triggerAtTime = SystemClock.elapsedRealtime() + Minutes;
		//此处设置开启AlarmReceiver这个Service
		Intent i = new Intent(this, AlarmReceiver.class);
		PendingIntent pi = PendingIntent.getBroadcast(this, 0, i, 0);
		//ELAPSED_REALTIME_WAKEUP表示让定时任务的出发时间从系统开机算起，并且会唤醒CPU。
		manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtTime, pi);
		return super.onStartCommand(intent, flags, startId);
	}

	@Override
	public void onDestroy() {
		super.onDestroy();

		//在Service结束后关闭AlarmManager
		AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
		Intent i = new Intent(this, AlarmReceiver.class);
		PendingIntent pi = PendingIntent.getBroadcast(this, 0, i, 0);
		manager.cancel(pi);

	}
}
</code></pre>
 
 
 
 AlarmReceiver:
 
 <pre><code class="language-java">import android.app.Notification;
import android.app.NotificationManager;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;

public class AlarmReceiver extends BroadcastReceiver {

	@Override
	public void onReceive(Context context, Intent intent) {
		//设置通知内容并在onReceive()这个函数执行时开启
		NotificationManager manager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
		Notification notification=new Notification(R.drawable.ic_launcher,"用电脑时间过长了！白痴！"
		,System.currentTimeMillis());
		notification.setLatestEventInfo(context, "快去休息！！！",
				"一定保护眼睛,不然遗传给孩子，老婆跟别人跑啊。", null);
		notification.defaults = Notification.DEFAULT_ALL;
		manager.notify(1, notification);


		//再次开启LongRunningService这个服务，从而可以
		Intent i = new Intent(context, LongRunningService.class);
		context.startService(i);
	}


}</code></pre>
 
 
 
 activity_main:
 
 <pre><code class="language-html">&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="15dp"
    android:orientation="vertical"
    &gt;

    &lt;TextView
        android:layout_marginBottom="20dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="护眼定时提醒"
        android:textSize="30sp"
        android:gravity="center_horizontal"
        /&gt;
    

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="提醒间隔时间:"
        android:textSize="25sp"
        /&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="90分钟"
        android:textSize="25sp"
        /&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="提醒音乐："
        android:textSize="25sp"
        /&gt;
    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="系统默认音乐"
        android:textSize="25sp"
        /&gt;
&lt;/LinearLayout&gt;</code></pre>
 
 
 
 千万不要忘了在AndroidManifest中注册Service！
 AndroidManifest:
 
 <pre><code class="language-html">&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.servicebestpractice"
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
            android:name="com.example.servicebestpractice.MainActivity"
            android:label="@string/app_name" &gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;

        &lt;service android:name=".LongRunningService" &gt;
        &lt;/service&gt;

        &lt;receiver android:name=".AlarmReceiver" &gt;
        &lt;/receiver&gt;
    &lt;/application&gt;

&lt;/manifest&gt;</code></pre>
 
 
 
 为了各位读者方便，在此笔者也是上传了源码：
 
 
 
 
 **此处有个不得不提的注意点**，笔者原来的代码是在Activity开启的时候自动开启Service，在Activity摧毁的时候自动摧毁Service，看上去好像可以运行，没有什么错误，并且在**10分钟内**的提醒基本都能够正常运行。
 但是倘若在比较长的时间提醒的时候就会出现**不提醒**的问题了！为什么呢，因为**android为了优化内存，减少耗电**，是会**自动清理内存**的，**会把后台的Service给清理掉。**
 至于具体怎么优化，还是暂且给读者们一个自我学习的空间吧。

