#android短信加密（发送加密短信，解密本地短信）
**短信加密**此类功能由于新手学习的需求量较小，所以在网上很少有一些简单的demo供新手参考。笔者做到此处也是花了比较多的时间自我构思，具体的过程也是不过多描述了，讲一下demo的内容。**（源码在文章结尾）**

 

<img alt="" class="has" height="530" src="https://img-blog.csdn.net/20151225110357255?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300"> <img alt="" class="has" height="530" src="https://img-blog.csdn.net/20151225110404947?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300"> <img alt="" class="has" height="530" src="https://img-blog.csdn.net/20151225110731080?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300">

<img alt="" class="has" height="530" src="https://img-blog.csdn.net/20151225110418897?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300"> <img alt="" class="has" height="530" src="https://img-blog.csdn.net/20151225110435367?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300">

 

demo功能：

1、可以发送短信并且加密（通过改变string中的char）

2、能够查看手机中的短信

3、能够给收到的加密短信解密。

 

涉及到的知识点：

1、intent bundle传递

2、ContentResolver获取手机短信

3、listveiw与simpleAdapter

4、发送短信以及为发送短信设置要监听的广播

 

遇到的问题：

1、发送短信字符过长会导致发送失败

解决方法：设置发送每条短信为70个字以内。

原理：每条短信限制160字符以内，每个汉字是2个字符。平时我们发送短信几乎不限长度，是因为一旦超过了单条短信的长度，手机会自动分多条发送，然后接收方分多条接收后整合在一起显示。

 

 

**代码：**

<img alt="" class="has" src="https://img-blog.csdn.net/20151225111803161?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

**MainActivity:**

 

```
import android.app.Activity;
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        InitView();
    }

    private void InitView() {
        Button send=(Button)findViewById(R.id.bt_send);
        Button receive=(Button)findViewById(R.id.bt_receive);

        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent=new Intent(MainActivity.this,SendActivity.class);
                startActivity(intent);
            }
        });

        receive.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent=new Intent(MainActivity.this,ReceiveActivity.class);
                startActivity(intent);
            }
        });
    }
}

```

 

 

**SendActivity:**

 

```
import android.app.Activity;
import android.app.PendingIntent;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.telephony.SmsManager;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

/**
 * Created by 佳佳 on 2015/12/21.
 */
public class SendActivity extends Activity {

    private IntentFilter sendFilter;
    private SendStatusReceiver sendStatusReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_send);

        InitView();
    }

    private void InitView() {
        Button cancel = (Button) findViewById(R.id.cancel_edit);
        Button send = (Button) findViewById(R.id.send_edit);
        final EditText phone = (EditText) findViewById(R.id.phone_edit_text);
        final EditText msgInput = (EditText) findViewById(R.id.content_edit_text);

        //为发送短信设置要监听的广播
        sendFilter = new IntentFilter();
        sendFilter.addAction("SENT_SMS_ACTION");
        sendStatusReceiver = new SendStatusReceiver();
        registerReceiver(sendStatusReceiver, sendFilter);

        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(SendActivity.this, "加密发送中，请稍后...", Toast.LENGTH_SHORT).show();
                //接收edittext中的内容,并且进行加密
                //倘若char+8超出了表示范围，则把原字符发过去
                String address = phone.getText().toString();
                String content = msgInput.getText().toString();
                String contents = "";
                for (int i = 0; i &lt; content.length(); i++) {
                    try {
                        contents += (char) (content.charAt(i) + 8);
                    }catch (Exception e) {
                        contents += (char) (content.charAt(i));
                    }
                }

                //Log.i("hahaha",contents);

                //发送短信
                //并使用sendTextMessage的第四个参数对短信的发送状态进行监控
                SmsManager smsManager = SmsManager.getDefault();
                Intent sentIntent = new Intent("SENT_SMS_ACTION");
                PendingIntent pi = PendingIntent.getBroadcast(
                        SendActivity.this, 0, sentIntent, 0);
                smsManager.sendTextMessage(address, null,
                        contents.toString(), pi, null);
            }
        });

        cancel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                finish();
            }
        });
    }

    class SendStatusReceiver extends BroadcastReceiver {

        @Override
        public void onReceive(Context context, Intent intent) {
            if (getResultCode() == RESULT_OK) {
                //发送成功
                Toast.makeText(context, "Send succeeded", Toast.LENGTH_LONG)
                        .show();

                Intent intent1 = new Intent(SendActivity.this, ReceiveActivity.class);
                startActivity(intent1);
                finish();
            } else {
                //发送失败
                Toast.makeText(context, "Send failed", Toast.LENGTH_LONG)
                        .show();
            }
        }

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //在Activity摧毁的时候停止监听
        unregisterReceiver(sendStatusReceiver);
    }
}

```

**ReceiveActivity:**

 

 

```
import android.app.Activity;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ListView;
import android.widget.SimpleAdapter;
import android.widget.TextView;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Created by 佳佳 on 2015/12/21.
 */
public class ReceiveActivity extends Activity implements AdapterView.OnItemClickListener{
    private TextView Tv_address;
    private TextView Tv_body;
    private TextView Tv_time;
    private ListView listview;
    private List&lt;Map&lt;String, Object&gt;&gt; dataList;
    private SimpleAdapter simple_adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_receive);

        InitView();
    }

    @Override
    protected void onStart() {
        super.onStart();
        RefreshList();
    }


    private void InitView() {
        Tv_address = (TextView) findViewById(R.id.tv_address);
        Tv_body = (TextView) findViewById(R.id.tv_body);
        Tv_time = (TextView) findViewById(R.id.tv_time);
        listview = (ListView) findViewById(R.id.list_receive);
        dataList = new ArrayList&lt;Map&lt;String, Object&gt;&gt;();

        listview.setOnItemClickListener(this);
    }

    private void RefreshList() {
        //从短信数据库读取信息
        Uri uri = Uri.parse("content://sms/");
        String[] projection = new String[]{"address", "body", "date"};
        Cursor cursor = getContentResolver().query(uri, projection, null, null, "date desc");
        startManagingCursor(cursor);

        //此处为了简化代码提高效率，仅仅显示20条最近短信
        for (int i = 0; i &lt; 20; i++) {
            //从手机短信数据库获取信息
            if(cursor.moveToNext()) {
                String address = cursor.getString(cursor.getColumnIndex("address"));
                String body = cursor.getString(cursor.getColumnIndex("body"));
                long longDate = cursor.getLong(cursor.getColumnIndex("date"));
                //将获取到的时间转换为我们想要的方式
                SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
                Date d = new Date(longDate);
                String time = dateFormat.format(d);


                Map&lt;String, Object&gt; map = new HashMap&lt;String, Object&gt;();
                map.put("address", address);
                map.put("body", body+"body");
                map.put("time", time+" time");
                dataList.add(map);
            }
        }

        simple_adapter = new SimpleAdapter(this, dataList, R.layout.activity_receive_list_item,
                new String[]{"address", "body", "time"}, new int[]{
                R.id.tv_address, R.id.tv_body, R.id.tv_time});
        listview.setAdapter(simple_adapter);
    }

    @Override
    public void onItemClick(AdapterView&lt;?&gt; adapterView, View view, int i, long l) {
        //获取listview中此个item中的内容
        //content的内容格式如下
        //{body=[B@43c2da70body, address=+8615671562394address, time=2015-12-24 11:55:50time}
        String content = listview.getItemAtPosition(i) + "";
        String body = content.substring(content.indexOf("body=") + 5,
                content.indexOf("body,"));
        //Log.i("hahaha",body);
        String address = content.substring(content.indexOf("address=") + 8,
                content.lastIndexOf(","));
        //Log.i("hahaha",address);
        String time = content.substring(content.indexOf("time=") + 5,
                content.indexOf(" time}"));
        //Log.i("hahaha",time);

        //使用bundle存储数据发送给下一个Activity
        Intent intent=new Intent(ReceiveActivity.this,ReceiveActivity_show.class);
        Bundle bundle = new Bundle();
        bundle.putString("body", body);
        bundle.putString("address", address);
        bundle.putString("time", time);
        intent.putExtras(bundle);
        startActivity(intent);

    }
}

```

 ReceiveActivity_show:

 

 

```
import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;

/**
 * Created by 佳佳 on 2015/12/24.
 */
public class ReceiveActivity_show extends Activity {
    private TextView Address_show;
    private TextView Time_show;
    private TextView Early_body_show;
    private TextView Late_body_show;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_receive_show);

        InitView();
    }

    private void InitView() {

        Address_show = (TextView) findViewById(R.id.address_show);
        Time_show = (TextView) findViewById(R.id.time_show);
        Early_body_show = (TextView) findViewById(R.id.early_body_show);
        Late_body_show = (TextView) findViewById(R.id.late_body_show);

        //接收内容和id
        Bundle bundle = this.getIntent().getExtras();
        String body = bundle.getString("body");
        String time = bundle.getString("time");
        String address = bundle.getString("address");


        Address_show.setText(address);
        Early_body_show.setText(body);
        Time_show.setText(time);

        //对短信消息进行解密后显示在textview中
        //倘若char+8超出了表示范围，则直接按照原字符解析
        String real_content = "";
        for (int i = 0; i &lt; body.length(); i++) {
            try {
                char textchar=(char) (body.charAt(i) + 8);
                real_content += (char) (body.charAt(i) - 8);
            }catch (Exception e){
                real_content += (char) (body.charAt(i));
            }
        }
        Late_body_show.setText(real_content);
    }

}

```

 

 

activity_main:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;

&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#000"
        android:padding="12dp"
        android:text="加密短信"
        android:textColor="#fff"
        android:textSize="25sp"
        android:textStyle="bold" /&gt;
    &lt;Button
        android:layout_marginTop="120dp"
        android:id="@+id/bt_send"
        android:layout_width="200dp"
        android:layout_height="80dp"
        android:text="发送加密短信"
        android:layout_gravity="center"
        android:textSize="20dp"/&gt;
    &lt;Button
        android:id="@+id/bt_receive"
        android:layout_width="200dp"
        android:layout_height="80dp"
        android:layout_gravity="center"
        android:text="解密本地短信"
        android:textSize="20dp"
        android:layout_below="@+id/bt_send"/&gt;

&lt;/LinearLayout&gt;
```

 activity_send:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;

&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;


    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="10dp"&gt;

        &lt;TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="加密短信发送"
            android:textColor="#000"
            android:textSize="35sp" /&gt;

        &lt;View
            android:layout_width="match_parent"
            android:layout_height="3dp"
            android:layout_marginBottom="20dp"
            android:background="#CECECE" /&gt;

        &lt;TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="5dp"
            android:text="发送至："
            android:textSize="25sp" /&gt;

        &lt;EditText
            android:id="@+id/phone_edit_text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="10dp"
            android:background="@drawable/edit_text_style"
            android:hint="接收人手机号码"
            android:maxLength="15"
            android:maxLines="1"
            android:textSize="19sp"
            android:singleLine="true" /&gt;

        &lt;TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="5dp"
            android:text="发送内容"
            android:textSize="25sp" /&gt;

        &lt;EditText
            android:id="@+id/content_edit_text"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:background="@drawable/edit_text_style"
            android:gravity="start"
            android:hint="单条短信请保持在70字以内"
            android:maxLength="70"
            android:textSize="19sp"
             /&gt;

        &lt;LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="10dp"&gt;

            &lt;Button
                android:id="@+id/cancel_edit"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="取消编辑"
                android:textSize="20sp" /&gt;

            &lt;Button
                android:id="@+id/send_edit"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="发 送"
                android:textSize="20sp" /&gt;
        &lt;/LinearLayout&gt;
    &lt;/LinearLayout&gt;
&lt;/LinearLayout&gt;
```

 activity_receive:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent"&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="10dp"
        android:text="所有短信"
        android:textColor="#fff"
        android:background="#000"
        android:textSize="23sp"
        android:textStyle="bold"/&gt;

    &lt;ListView
        android:id="@+id/list_receive"
        android:layout_width="match_parent"
        android:layout_height="match_parent"&gt;&lt;/ListView&gt;

&lt;/LinearLayout&gt;
```

 activity_receive_show:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;

&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#000"
        android:padding="12dp"
        android:text="短信解密"
        android:textColor="#fff"
        android:textSize="25sp"
        android:textStyle="bold" /&gt;

    &lt;TableLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="10dp"
        android:stretchColumns="1"
        android:shrinkColumns="1"&gt;

        &lt;TableRow android:layout_marginTop="10dp"
            android:layout_width="match_parent"
            &gt;

            &lt;TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginBottom="5dp"
                android:text="号码："
                android:textColor="#000"
                android:textSize="20sp" /&gt;

            &lt;TextView
                android:id="@+id/address_show"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginBottom="10dp"
                android:background="@drawable/edit_text_style"
                android:maxLines="1"
                android:singleLine="true"
                android:textSize="18sp" /&gt;
        &lt;/TableRow
            &gt;

        &lt;TableRow
            android:layout_width="match_parent"
            android:layout_marginBottom="10dp"
            &gt;

            &lt;TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginBottom="5dp"
                android:text="时间："
                android:textColor="#000"
                android:textSize="20sp" /&gt;

            &lt;TextView
                android:id="@+id/time_show"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginBottom="10dp"
                android:background="@drawable/edit_text_style"
                android:maxLines="1"
                android:textSize="18sp" /&gt;
        &lt;/TableRow&gt;

        &lt;TableRow android:layout_marginBottom="10dp"
            android:layout_width="match_parent"&gt;

            &lt;TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginBottom="5dp"
                android:text="原文本："
                android:textColor="#000"
                android:textSize="20sp" /&gt;

            &lt;TextView
                android:id="@+id/early_body_show"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="@drawable/edit_text_style"
                android:minLines="6"
                android:maxLines="6"
                android:textSize="18sp"
                /&gt;
        &lt;/TableRow&gt;

        &lt;TableRow
            android:layout_width="match_parent"&gt;

            &lt;TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginBottom="5dp"
                android:text="解析后："
                android:textColor="#000"
                android:textSize="20sp" /&gt;

            &lt;TextView
                android:id="@+id/late_body_show"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="@drawable/edit_text_style"
                android:minLines="6"
                android:maxLines="6"
                android:singleLine="false"
                android:textSize="18sp"
                /&gt;
        &lt;/TableRow&gt;


    &lt;/TableLayout&gt;
&lt;/LinearLayout&gt;
```

 

 源码地址：

 

 

 

 

 

 
