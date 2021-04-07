#android 自定义AlertDialog 与Activity相互传递数据
**主要实现功能：**

1、从Activity的TextView中获取字符串设置到AlertDialog的TextView和EditText中

2、将AlertDialog的EditText中的值设置到Activity的TextView中

 

效果：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/420.png" width="300" height="500" alt="">  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/421.png" width="300" height="500" alt="">  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/422.png" width="300" height="500" alt=""> 

 

新手在自定义AlertDialog上的疑问笔者猜测主要有**两个**：

1、自定义的layout如何放到AlertDialog中？

解答：

获取到layout的view之后，直接调用AlertDialog.Builder的setView方法即可。

 

2、如何对自定义AlertDialog中的控件进行操作？

解答：

于fragment中的操作类似，首先要获取该layout的view，然后通过该view获取到其中控件进行操作。

 

 

MainActivity：



```
package com.example.myalertdialog;

import android.app.Activity;
import android.content.DialogInterface;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends Activity {

    TextView old_name;
    Button bt_change_name;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        old_name = (TextView) findViewById(R.id.tv_name);
        bt_change_name = (Button) findViewById(R.id.bt_name);

        bt_change_name.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //获取自定义AlertDialog布局文件的view
                LinearLayout change_name = (LinearLayout) getLayoutInflater()
                        .inflate(R.layout.my_dialog, null);
                TextView tv_name_dialog = (TextView) change_name.findViewById(R.id.tv_name_dialog);
                //由于EditText要在内部类中对其进行操作，所以要加上final
                final EditText et_name_dialog = (EditText) change_name.findViewById(R.id.et_name_dialog);

                //设置AlertDialog中TextView和EditText显示Activity中TextView的内容
                tv_name_dialog.setText(old_name.getText().toString());
                et_name_dialog.setText(old_name.getText().toString());
                new AlertDialog.Builder(MainActivity.this)
                        .setTitle("修改用户名")
                        .setView(change_name)
                        .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                //将Activity中的textview显示AlertDialog中EditText中的内容
                                //并且用Toast显示一下
                                old_name.setText(et_name_dialog.getText().toString());
                                Toast.makeText(MainActivity.this, "设置成功！", Toast.LENGTH_SHORT).show();
                            }
                        })
                        //由于“取消”的button我们没有设置点击效果，直接设为null就可以了
                        .setNegativeButton("取消", null)
                        .create()
                        .show();

            }
        });


    }
}

```



activity_main.xml:



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="10dp"&gt;

&lt;LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"&gt;
    &lt;TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="#000"
        android:text="原用户名：" /&gt;
    &lt;TextView
        android:id="@+id/tv_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="大西瓜" /&gt;
&lt;/LinearLayout&gt;

   &lt;Button
       android:id="@+id/bt_name"
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       android:text="修改用户名"
       /&gt;


&lt;/LinearLayout&gt;

```



 



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="10dp"&gt;

&lt;LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"&gt;
    &lt;TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="#000"
        android:text="原用户名：" /&gt;
    &lt;TextView
        android:id="@+id/tv_name_dialog"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Hello World!" /&gt;
&lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"&gt;
        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="#000"
            android:text="新用户名" /&gt;
        &lt;EditText
            android:id="@+id/et_name_dialog"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            /&gt;
    &lt;/LinearLayout&gt;


&lt;/LinearLayout&gt;

```



 

 

 

 