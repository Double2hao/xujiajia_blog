#android中简单的Handler与Message
前提概要

虽然笔者已经学习了android的AsyncTask来实现一部消息的处理。但是在android的学习中，经常会在一些demo中看到Handler与Message的一些使用，所以Handler与Message的学习也是有必要了。至于学多少，笔者还是比较坚持自己的看法，“用多少，学多少”，毕竟已经有了AsyncTask如此方便的东西，Handler与Message也不是那么必不可缺了。(如此文的简单了解一下还是不需要花太多时间的)

 

此实例是在handler中更新textview的内容，新手读者可能会问为什么不直接在oncreate中一行解决呢？还是主要是需求问题，倘若我们需要在子线程中从网上获取内容，然后更新到textview中，那么直接写在主线程中不合理的。但是由于此实例主要是一个参考作用，并且让它更能让新手理解，所以并没有写从网络获取内容的代码了。（直接在子线程中是不能执行更新UI的操作的，程序会崩溃）

 

上一下效果图：

分别是点击button前后效果

 

<img alt="" class="has" height="700" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039739320.png" width="400">    <img alt="" class="has" height="700" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039741281.png" width="400">

 

 

好了，直接看一下概念与代码：

 

**Message:**

Message是在线程之间传递的消息，它可以在内部携带少量的信息，用于再不同线程之间交换数据。除了what字段，还可以用arge1和arg2字段来携带一些整型数据，使用obj字段携带一个Object对象。

 

**Handler**

Handler顾名思义就是处理者的意思，它主要是用于发送和处理消息的。发送消息一般是使用Handler的sendMessage()方法，而发出的消息经过一系列地辗转处理后，最终会传递到Handler的handleMessage()方法中。

 

 

MainActivity:

 

```
import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends Activity implements OnClickListener {

	//定义UPDATE_TEXT这个整型敞亮，用于表示更新TextView这个动作
	public static final int UPDATE_TEXT = 1;

	private TextView text;
	private Button changeText;

	//创建一个Handler
	private Handler handler = new Handler() {

		public void handleMessage(Message msg) {
			switch (msg.what) {
			case UPDATE_TEXT:
				//在这里可以进行UI操作
				//对msg.obj进行String强制转换
				String string=(String)msg.obj;
				text.setText(string);
				break;
			default:
				break;
			}
		}

	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		text = (TextView) findViewById(R.id.text);
		changeText = (Button) findViewById(R.id.change_text);
		changeText.setOnClickListener(this);
	}

	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.change_text:
			new Thread(new Runnable() {
				@Override
				public void run() {
					//创建一个message
					//设置what字段的值为UPDATE_TEXT,主要是为了区分不同的message
					//设置message.obj的内容
					//调用Handler的message对象
					//handler中的handlermessage对象是在主线程中运行的
					String string="Nice to meet you";
					Message message = new Message();
					message.what = UPDATE_TEXT;
					message.obj=string;
					handler.sendMessage(message);
				}
			}).start();
			break;
		default:
			break;
		}
	}

}
```

 

 

 

activity_main:

 

```
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" &gt;

    &lt;Button
        android:id="@+id/change_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Change Text" /&gt;

    &lt;TextView
        android:id="@+id/text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="Hello world"
        android:textSize="20sp" /&gt;

&lt;/RelativeLayout&gt;
```

 

 

 

 

 

 