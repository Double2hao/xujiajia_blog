#Toast,popupWindow,AlertDialog
# 前提概要

提示框是我们经常会使用的功能，Toast,popupWindow,AlertDialog这三种是我们用的较多的，甚至在有些情况下使用任何一个都是可以的。本文主要分析下它们各自的优点、用法、以及适用场景。

# 效果

本文主要模仿微信小程序的dialog为例子。（为了代码的简略，就不添加圆角等效果了）  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040212660.png" alt="这里写图片描述" title="">

笔者用Toast,popupWindow,AlertDialog三者实现的效果如下：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040213261.png" alt="这里写图片描述" title=""><img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040214242.png" alt="这里写图片描述" title=""><img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040215623.png" alt="这里写图片描述" title="">

为了显示正确，在布局最外面笔者套了一层FrameLayout，如果不套FrameLayout的显示效果如下：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040216304.png" alt="这里写图片描述" title=""><img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040217695.png" alt="这里写图片描述" title=""><img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040218876.png" alt="这里写图片描述" title="">

>  
 上面6张图的比较可以看出，如果在layout最外层不嵌套一个ViewGroup，最终宽高会忽视layout父View的设置而直接根据子View内容的大小绘制，那么layout可能会达不到我们预期的显示。 
 所以笔者如果有使用的需求，一定不要在最外层嵌套一层ViewGroup。 


# 优劣分析

## Toast
1. 集成度很高，使用超级简单。1. 不可以长时间显示，显示时间最长为Toast.LENGTH_SHORT。不适合作为网络获取的提示。1. 由于可控性较低，所以坑比较少。
## popupWindow
1. 和Toast使用效果几乎一样，但是集成度相对较低，可控性更高。1. 显示时间可以自己控制，使用handler机制就可以。1. 真正使用的时候会发现坑比较多。比如layout外面必须嵌套一层viewGroup;background要设置为null，不然会有阴影等。
## AlertDialog
1. 使用形状有很大的限制，宽度基本是写死的。1. 如果用来显示text，title之类的常规信息非常实用，不需要自己订制view，官方集成的非常好。
# 代码

**MainActivity.java**

```
package com.example.xujiajia_sx.dialogtest;

import android.os.Bundle;
import android.os.Handler;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.view.Gravity;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.Button;
import android.widget.PopupWindow;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {<!-- -->

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        Button btnToast = findViewById(R.id.btn_toast);

        btnToast.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast toast=new Toast(MainActivity.this);
                View layout= LayoutInflater.from(MainActivity.this).inflate(R.layout.test_view,null,false);
                toast.setView(layout);
                toast.setGravity(Gravity.CENTER,0,0);
                toast.setDuration(Toast.LENGTH_SHORT);
                toast.show();
            }
        });
        Button btnPopupWindow = findViewById(R.id.btn_popupWindow);
        btnPopupWindow.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                final PopupWindow popupWindow=new PopupWindow(MainActivity.this);
                View layout= LayoutInflater.from(MainActivity.this).inflate(R.layout.test_view,null,false);
                popupWindow.setContentView(layout);
                popupWindow.setBackgroundDrawable(null);
                popupWindow.showAtLocation(getWindow().getDecorView(),Gravity.CENTER,0,0);
                new Handler().postDelayed(new Runnable() {
                    @Override
                    public void run() {
                        popupWindow.dismiss();
                    }
                },2000);
            }
        });
        Button btnAlertDialog = findViewById(R.id.btn_alertDialog);
        btnAlertDialog.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                AlertDialog.Builder builder=new AlertDialog.Builder(MainActivity.this);
                View layout= LayoutInflater.from(MainActivity.this).inflate(R.layout.test_view,null,false);
                builder.setView(layout);
                builder.create().show();
            }
        });
    }
}

```

**activity_main.xml**

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.example.xujiajia_sx.dialogtest.MainActivity"&gt;

    &lt;Button
        android:id="@+id/btn_toast"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/toast" /&gt;
    &lt;Button
        android:id="@+id/btn_popupWindow"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/popupWindow" /&gt;
    &lt;Button
        android:id="@+id/btn_alertDialog"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/alertDialog" /&gt;

&lt;/LinearLayout&gt;

```

**test_view.xml**

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;

&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/ll_toast"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:alpha="0.5"
    android:background="#000000"
    android:orientation="vertical"
    android:paddingBottom="16dp"
    android:paddingLeft="8dp"
    android:paddingRight="8dp"
    android:paddingTop="8dp"&gt;

    &lt;RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="8dp"
        android:layout_weight="1"&gt;

        &lt;ProgressBar
            android:id="@+id/pb_toast"
            android:layout_width="25dp"
            android:layout_height="25dp"
            android:layout_centerInParent="true" /&gt;
    &lt;/RelativeLayout&gt;

    &lt;TextView
        android:id="@+id/tv_toast"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:maxLines="1"
        android:text="@string/show_text"
        android:textColor="#ffffff" /&gt;
&lt;/LinearLayout&gt;

```