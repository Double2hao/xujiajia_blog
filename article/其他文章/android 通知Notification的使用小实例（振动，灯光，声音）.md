#android 通知Notification的使用小实例（振动，灯光，声音）
效果图：

>  
 <img alt="" class="has" height="800" src="https://img-blog.csdn.net/20151026110151715?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="450"> 
   
 <img alt="" class="has" height="800" src="https://img-blog.csdn.net/20151026110159471?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="450"> 
   
 <img alt="" class="has" height="800" src="https://img-blog.csdn.net/20151026110244179?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="450"> 
   
   
 MainActivity: 
   
 <pre class="has"><code class="language-java">import java.io.File;

import android.app.Activity;
import android.app.Notification;
import android.app.NotificationManager;
import android.app.PendingIntent;
import android.content.Intent;
import android.graphics.Color;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity implements OnClickListener {

	private Button sendNotice;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		sendNotice = (Button) findViewById(R.id.send_notice);
		sendNotice.setOnClickListener(this);
	}

	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.send_notice:
			NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);

			//创建notification对象来存储通知所需的各种信息
			//第一个参数为图标
			//第二个参数用于指定通知的ticker内容
			//第三个参数用于指定通知被创建的时间，以毫秒为单位
			Notification notification = new Notification(
					R.drawable.ic_launcher, "This is ticker text",
					System.currentTimeMillis());

			//此处设置点击的activity的跳转
			//第一个参数依旧是Context
			//第二个参数一般用不到，所以用0表示取默认值
			//第三个参数就是一个Intent对象
			//FLAG_CANCEL_CURRENT:如果当前系统中已经存在一个相同的PendingIntent对象，
			// 那么就将先将已有的PendingIntent取消，然后重新生成一个PendingIntent对象。
			Intent intent = new Intent(this, NotificationActivity.class);
			PendingIntent pi = PendingIntent.getActivity(this, 0, intent,
					PendingIntent.FLAG_CANCEL_CURRENT);

			//设置通知的布局
			//第一个参数为Context
			//第二个参数用于指定通知的标题
			//第三个参数用于指定通知的征文内容
			//第四个参数用于传入PendingIntent对象，用于设置点击效果
			notification.setLatestEventInfo(this, "This is content title",
					"This is content text", pi);

//			//设置在通知发出的时候的音频
//			Uri soundUri = Uri.fromFile(new File("/system/media/audio/ringtones/Basic_tone.ogg"));
//			notification.sound = soundUri;
//
//			//设置手机震动
//			//第一个，0表示手机静止的时长，第二个，1000表示手机震动的时长
//			//第三个，1000表示手机震动的时长，第四个，1000表示手机震动的时长
//			//此处表示手机先震动1秒，然后静止1秒，然后再震动1秒
//			long[] vibrates = {0, 1000, 1000, 1000};
//			notification.vibrate = vibrates;
//
//			//设置LED指示灯的闪烁
//			//ledARGB设置颜色
//			//ledOnMS指定LED灯亮起的时间
//			//ledOffMS指定LED灯暗去的时间
//			//flags用于指定通知的行为
//			notification.ledARGB = Color.GREEN;
//			notification.ledOnMS = 1000;
//			notification.ledOffMS = 1000;
//			notification.flags = Notification.FLAG_SHOW_LIGHTS;

			//如果不想进行那么多繁杂的这只，可以直接使用通知的默认效果
			//默认设置了声音，震动和灯光
			notification.defaults = Notification.DEFAULT_ALL;

			//使用notify将通知显示出来
			//第一个参数是id，要爆炸为每个通知所指定的id是不同的
			//第二个参数就是Notification对象
			manager.notify(1, notification);
			break;
		default:
			break;
		}
	}

}
</code></pre>   
   
 activity_main: 
   
 <pre class="has"><code class="language-html">&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" &gt;

	&lt;Button 
	    android:id="@+id/send_notice"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:text="发出通知"
	    /&gt;
    
&lt;/LinearLayout&gt;</code></pre> 
   
   
   
 NotificationActivity: 
   
 <pre class="has"><code class="language-java">import android.app.Activity;
import android.app.NotificationManager;
import android.os.Bundle;

public class NotificationActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.notification_layout);

		//打开NotificationActivity这个Activity后把通知给关掉
		NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
		manager.cancel(1);
	}

}</code></pre>   
   
 notification_layout: 
   
 <pre class="has"><code class="language-html">&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" &gt;
	
    &lt;TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:textSize="24sp"
        android:text="这是通知点击后的界面"
        /&gt;
    
&lt;/RelativeLayout&gt;</code></pre>   
   
 AndroidManifest: 
   
 <pre class="has"><code class="language-html">&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.notificationtest"
    android:versionCode="1"
    android:versionName="1.0" &gt;

    &lt;uses-sdk
        android:minSdkVersion="14"
        android:targetSdkVersion="19" /&gt;

    &lt;application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" &gt;
        &lt;activity
            android:name="com.example.notificationtest.MainActivity"
            android:label="@string/app_name" &gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
        &lt;activity android:name=".NotificationActivity" &gt;
        &lt;/activity&gt;

    &lt;/application&gt;

&lt;/manifest&gt;</code></pre> 
   
   
   最后附上源码： 
  
   


 