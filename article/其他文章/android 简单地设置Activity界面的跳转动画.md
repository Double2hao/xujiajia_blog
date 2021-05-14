#android 简单地设置Activity界面的跳转动画
动画这一知识点算是水比较深了，主要在自定义动画中可是大有文章，并且技术都会了后就需要看设计能力了。

当然这些不是笔者博客的重点，笔者还是基本只讲技术上的，本篇博客就讲一讲简单的**设置Activity的跳转动画**。（其实就是调用一些系统内置的动画，暂时不涉及自己写动画。）

 

效果：（代码其实很简单，就不**上传**源码了。）

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040232230.png">

 

**可能碰到的问题：**

在输入“R.anim.”之后没有自动提示，Control+鼠标左击“R.anim”,然后可以看到系统内置的一些动画，直接复制黏贴即可。如下图：

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040233771.png">

 

系统内置动画：（其他的一些读者可以自己试试看）

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040234342.png">

 

（有好奇的读者可能希望自己也写一个动画，在下一篇博客，笔者会专门写。）

代码如下：**（android版本需要在3.0以上）**

MainActivity：

 

```
package com.example.animationchanges;

import android.app.Activity;
import android.app.ActivityOptions;
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.view.animation.AlphaAnimation;
import android.widget.Button;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button button=(Button)findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new Intent(MainActivity.this,OneActivity.class);
                startActivity(intent);
                //设置跳转动画
                overridePendingTransition(R.anim.abc_slide_in_bottom,R.anim.abc_slide_out_bottom);

            }


        });
    }
}

```

 

 

 

 

 

OneActivity：

 

```
package com.example.animationchanges;

import android.app.Activity;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;

public class OneActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_one);
    }
}

```

 

 

activity_main:

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#a31212"
    &gt;

    &lt;Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="界面切换"/&gt;

&lt;/RelativeLayout&gt;

```

 

activity_one:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#1f9c16"
    &gt;


&lt;/RelativeLayout&gt;

```

 

 

 

 

 

 

 

 