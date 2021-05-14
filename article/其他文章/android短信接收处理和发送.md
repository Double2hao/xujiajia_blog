#android短信接收处理和发送
关于短信接收处理方面，当前已经有一些app做的比较好了，比如发给手机发验证码验证的问题，很多app在手机接收到验证码后，不需要输入，就直接可以跳过验证界面，这就是用到了对接收到的短信的处理。至于短信的发送，也没什么好说的了。在此也只是附上一个小实例。

 

效果图：

<img alt="" class="has" height="800" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1970.png" width="450">

 

 

MainActivity:

 

```
import android.app.Activity;
import android.app.PendingIntent;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.telephony.SmsManager;
import android.telephony.SmsMessage;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends Activity {

	private TextView sender;
	private TextView content;
	private IntentFilter receiveFilter;
	private MessageReceiver messageReceiver;


	private EditText to;
	private EditText msgInput;
	private Button send;
	private IntentFilter sendFilter;
	private SendStatusReceiver sendStatusReceiver;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		sender = (TextView) findViewById(R.id.sender);
		content = (TextView) findViewById(R.id.content);
		to = (EditText) findViewById(R.id.to);
		msgInput = (EditText) findViewById(R.id.msg_input);
		send = (Button) findViewById(R.id.send);

		//为接收短信设置要监听的广播
		receiveFilter = new IntentFilter();
		receiveFilter.addAction("android.provider.Telephony.SMS_RECEIVED");
		messageReceiver = new MessageReceiver();
		registerReceiver(messageReceiver, receiveFilter);

		//为发送短信设置要监听的广播
		sendFilter = new IntentFilter();
		sendFilter.addAction("SENT_SMS_ACTION");
		sendStatusReceiver = new SendStatusReceiver();
		registerReceiver(sendStatusReceiver, sendFilter);

		send.setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				//发送短信
				//并使用sendTextMessage的第四个参数对短信的发送状态进行监控
				SmsManager smsManager = SmsManager.getDefault();
				Intent sentIntent = new Intent("SENT_SMS_ACTION");
				PendingIntent pi = PendingIntent.getBroadcast(
						MainActivity.this, 0, sentIntent, 0);
				smsManager.sendTextMessage(to.getText().toString(), null,
						msgInput.getText().toString(), pi, null);
			}
		});
	}

	@Override
	protected void onDestroy() {
		super.onDestroy();
		//在Activity摧毁的时候停止监听
		unregisterReceiver(messageReceiver);
		unregisterReceiver(sendStatusReceiver);
	}

	class MessageReceiver extends BroadcastReceiver {

		@Override
		public void onReceive(Context context, Intent intent) {
			Bundle bundle = intent.getExtras();
			//使用pdu秘钥来提取一个pdus数组
			Object[] pdus = (Object[]) bundle.get("pdus");

			SmsMessage[] messages = new SmsMessage[pdus.length];
			for (int i = 0; i &lt; messages.length; i++) {
				messages[i] = SmsMessage.createFromPdu((byte[]) pdus[i]);
			}

			//获取发送方号码
			String address = messages[0].getOriginatingAddress();

			//获取短信内容
			String fullMessage = "";
			for (SmsMessage message : messages) {
				fullMessage += message.getMessageBody();
			}
			sender.setText(address);
			content.setText(fullMessage);
		}

	}

	class SendStatusReceiver extends BroadcastReceiver {

		@Override
		public void onReceive(Context context, Intent intent) {
			if (getResultCode() == RESULT_OK) {
				//发送成功
				Toast.makeText(context, "Send succeeded", Toast.LENGTH_LONG)
						.show();
			} else {
				//发送失败
				Toast.makeText(context, "Send failed", Toast.LENGTH_LONG)
						.show();
			}
		}

	}

}
```

 

 

 

 

 

activity_main:

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" &gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="50dp" &gt;

        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:padding="10dp"
            android:text="From:" /&gt;

        &lt;TextView
            android:id="@+id/sender"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="50dp" &gt;

        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:padding="10dp"
            android:text="Content:" /&gt;

        &lt;TextView
            android:id="@+id/content"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="50dp" &gt;

        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:padding="10dp"
            android:text="To:" /&gt;

        &lt;EditText
            android:id="@+id/to"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:layout_weight="1" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="50dp" &gt;

        &lt;EditText
            android:id="@+id/msg_input"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:layout_weight="1" /&gt;

        &lt;Button
            android:id="@+id/send"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:text="Send" /&gt;
    &lt;/LinearLayout&gt;

&lt;/LinearLayout&gt;
```

 

 

 

 

 

AndroidManifest:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.smstest"
    android:versionCode="1"
    android:versionName="1.0" &gt;

    &lt;uses-sdk
        android:minSdkVersion="14"
        android:targetSdkVersion="17" /&gt;

    //接受短信
    &lt;uses-permission android:name="android.permission.RECEIVE_SMS" /&gt;
    //发送短信
    &lt;uses-permission android:name="android.permission.SEND_SMS" /&gt;

    &lt;application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" &gt;
        &lt;activity
            android:name="com.example.smstest.MainActivity"
            android:label="@string/app_name" &gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
    &lt;/application&gt;

&lt;/manifest&gt;
```

 

 

 

 

 

 