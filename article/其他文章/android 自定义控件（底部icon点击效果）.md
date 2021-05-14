#android 自定义控件（底部icon点击效果）


此片文章算是笔者之前写的一篇自定义控件的扩展，此片文章觉得吃力的可以先看前一篇，原文地址：



 

另外，笔者此篇中的功能一般会搭配fragment一起使用，笔者介绍fragment的地址如下：

#  



 

效果：**（源码在文章结尾）**

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039368220.png" width="300" height="450" alt=""> 

 

**主要实现的功能：**

1、在java代码中动态设置底部控件的icon和text。（搭配fragment的时候特别方便）

2、text点击时会加粗

3、封装成自定义控件，更加方便。

(考虑到新手可能不易理解，笔者代码没有多加功能) 

 

**把底部icon做成自定义控件的优势：**

1、搭配fragment或者viewpager使用的时候更加方便，避免写过多重复性代码。

2、代码维护起来更加方便，比如要修改底部icon中的字体，直接在自定义控件的layout中修改就可以。

3、提高代码的可阅读性。

 

代码截图：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039368781.png" alt=""> 

 

MainActivity:



```
package com.example.double2.mybottomlayout;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends Activity implements View.OnClickListener {

    BottomLayout blSituation;
    BottomLayout blMap;
    BottomLayout blDiscover;
    BottomLayout blMyData;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.act_main);

        //初始化控件
        initView();
    }

    private void initView() {
        initBottomLayout();

    }

    private void initBottomLayout() {
        blSituation = (BottomLayout) findViewById(R.id.bottom_icon_situation);
        blMap = (BottomLayout) findViewById(R.id.bottom_icon_map);
        blDiscover = (BottomLayout) findViewById(R.id.bottom_icon_discover);
        blMyData = (BottomLayout) findViewById(R.id.bottom_icon_my_data);

        blSituation.setNormalIcon(R.drawable.bottom_icon_situation_normal);
        blSituation.setFocusIcon(R.drawable.bottom_icon_situation_focus);
        blSituation.setIconText("Situation");
        blSituation.setFocused(true);
        blSituation.setOnClickListener(this);

        blMap.setNormalIcon(R.drawable.bottom_icon_map_normal);
        blMap.setFocusIcon(R.drawable.bottom_icon_map_focus);
        blMap.setIconText("Map");
        blMap.setFocused(false);
        blMap.setOnClickListener(this);

        blDiscover.setNormalIcon(R.drawable.bottom_icon_discover_normal);
        blDiscover.setFocusIcon(R.drawable.bottom_icon_discover_focus);
        blDiscover.setIconText("Discover");
        blDiscover.setFocused(false);
        blDiscover.setOnClickListener(this);

        blMyData.setNormalIcon(R.drawable.bottom_icon_my_data_normal);
        blMyData.setFocusIcon(R.drawable.bottom_icon_my_data_focus);
        blMyData.setIconText("MyData");
        blMyData.setFocused(false);
        blMyData.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.bottom_icon_situation:
                blSituation.setFocused(true);
                blMap.setFocused(false);
                blDiscover.setFocused(false);
                blMyData.setFocused(false);
                break;
            case R.id.bottom_icon_map:
                blSituation.setFocused(false);
                blMap.setFocused(true);
                blDiscover.setFocused(false);
                blMyData.setFocused(false);
                break;
            case R.id.bottom_icon_discover:
                blSituation.setFocused(false);
                blMap.setFocused(false);
                blDiscover.setFocused(true);
                blMyData.setFocused(false);
                break;
            case R.id.bottom_icon_my_data:
                blSituation.setFocused(false);
                blMap.setFocused(false);
                blDiscover.setFocused(false);
                blMyData.setFocused(true);
                break;
        }
    }
}

```



 



```
package com.example.double2.mybottomlayout;

import android.content.Context;
import android.graphics.Typeface;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.TextView;

/**
 * Created by Double2号 on 2016/4/8.
 */
public class BottomLayout extends LinearLayout {

    private int normalIcon;
    private int focusIcon;
    private boolean isFocused=false;
    private ImageView ivIcon;
    private TextView tvText;

    public BottomLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        //加载布局文件，与setContentView()效果一样
        LayoutInflater.from(context).inflate(R.layout.act_main_bottom_layout, this);
        ivIcon = (ImageView) findViewById(R.id.iv_main_bottom_icon);
        tvText = (TextView) findViewById(R.id.tv_main_bottom_text);
    }

    public void setNormalIcon(int normalIcon) {
        this.normalIcon = normalIcon;
        ivIcon.setImageResource(normalIcon);
    }

    public void setFocusIcon(int focusIcon) {
        this.focusIcon = focusIcon;
    }

    public void setIconText(String text) {
        tvText.setText(text);
    }

    public void setFocused(boolean isFocused) {
        //如果已经处在设置的状态中，就不进行操作
        if (this.isFocused != isFocused) {
            this.isFocused = isFocused;
            if (isFocused) {
                //设置获得焦点后的图片
                //文字加粗
                ivIcon.setImageResource(focusIcon);
                tvText.setTypeface(Typeface.defaultFromStyle(Typeface.BOLD));
            } else {
                //设置获得普通状态的图片
                //文字不加粗
                ivIcon.setImageResource(normalIcon);
                tvText.setTypeface(Typeface.defaultFromStyle(Typeface.NORMAL));
            }
        }
    }

}

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="0dp"
        android:layout_weight="9" /&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal"&gt;

        &lt;com.example.double2.mybottomlayout.BottomLayout
            android:id="@+id/bottom_icon_situation"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"&gt;&lt;/com.example.double2.mybottomlayout.BottomLayout&gt;


        &lt;com.example.double2.mybottomlayout.BottomLayout
            android:id="@+id/bottom_icon_map"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"&gt;

        &lt;/com.example.double2.mybottomlayout.BottomLayout&gt;&gt;

        &lt;com.example.double2.mybottomlayout.BottomLayout
            android:id="@+id/bottom_icon_discover"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"&gt;

        &lt;/com.example.double2.mybottomlayout.BottomLayout&gt;&gt;

        &lt;com.example.double2.mybottomlayout.BottomLayout
            android:id="@+id/bottom_icon_my_data"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"&gt;

        &lt;/com.example.double2.mybottomlayout.BottomLayout&gt;
    &lt;/LinearLayout&gt;

&lt;/LinearLayout&gt;

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center_horizontal"
    android:padding="5dp"
    android:background="#eae17b"&gt;

    &lt;ImageView
        android:id="@+id/iv_main_bottom_icon"
        android:layout_width="wrap_content"
        android:layout_height="0dp"
        android:layout_weight="1"
        tools:src="@mipmap/ic_launcher"/&gt;
    &lt;TextView
        android:id="@+id/tv_main_bottom_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@android:color/holo_blue_dark"
        android:textSize="10sp"
        tools:text="icon"/&gt;
&lt;/LinearLayout&gt;
```



 

 

 