#tabhost选项卡，做微信以及微博UI界面必学
# Android UI-实现切换标签（tabhost与FrameLayout）

 

  此篇主要是介绍tabhost的简单使用，比较实用，也比较容易学，本人在上传之前已经删除了大部分无用的代码，希望同样的学习者能更有效率并且有效地学习，觉得有需要的朋友可以直接拉到尾部下载源码。（刚刚开始写博客，不太知道如何阐述，见谅）

 

先展示一下效果图:

<img alt="" class="has" height="384" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/580.png" width="216"> <img alt="" class="has" height="384" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/581.png" width="216"> <img alt="" class="has" height="384" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/582.png" width="216"> 

 

然后直接代码吧：

 

**MainActivity.java**

 

```
package com.example.tabhost;

import android.app.Activity;
import android.os.Bundle;
import android.support.annotation.RequiresPermission;
import android.view.Menu;
import android.view.MenuItem;
import android.view.Window;
import android.widget.TabHost;

public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		requestWindowFeature(Window.FEATURE_NO_TITLE);
		setContentView(R.layout.activity_main);
		settabhost();
	}

	private void settabhost() {
		// TODO Auto-generated method stub
		TabHost tabhost = (TabHost) findViewById(android.R.id.tabhost);
		tabhost.setup();
		tabhost.addTab(tabhost
				.newTabSpec("tab1")
				.setIndicator(null, getResources().getDrawable(R.drawable.tab1))
				.setContent(R.id.setting_password));
		tabhost.addTab(tabhost
				.newTabSpec("tab2")
				.setIndicator(null, getResources().getDrawable(R.drawable.tab2))
				.setContent(R.id.setting_background));
		tabhost.addTab(tabhost
				.newTabSpec("tab3")
				.setIndicator(null, getResources().getDrawable(R.drawable.tab3))
				.setContent(R.id.setting_sound));
		tabhost.setCurrentTab(0);
	}
            
}
```

 

 

 

 

 

**activity_main.xml**

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" &gt;

    &lt;RelativeLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:background="#000000" &gt;

        &lt;Button
            android:id="@+id/backToMain"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="返回"
            android:textColor="#ffffff" /&gt;

        &lt;Button
            android:id="@+id/toRelation"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:text="关于"
            android:textColor="#ffffff" /&gt;
    &lt;/RelativeLayout&gt;

    &lt;TabHost
        android:id="@android:id/tabhost"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent" &gt;

        &lt;!-- 此处为选项所在位置 --&gt;

        &lt;LinearLayout
            android:id="@+id/tab11"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@drawable/setbackground"
            android:orientation="vertical" &gt;

            &lt;TabWidget
                android:id="@android:id/tabs"
                android:layout_width="fill_parent"
                android:layout_height="wrap_content" &gt;
            &lt;/TabWidget&gt;

            &lt;FrameLayout
                android:id="@android:id/tabcontent"
                android:layout_width="fill_parent"
                android:layout_height="fill_parent" &gt;

                &lt;include layout="@layout/setting_password" /&gt;

                &lt;include layout="@layout/setting_background" /&gt;
"
                &lt;include layout="@layout/setting_sound" /&gt;
            &lt;/FrameLayout&gt;
        &lt;/LinearLayout&gt;
    &lt;/TabHost&gt;

&lt;/LinearLayout&gt;
```

 

 

 

 

**setting_background.xml**

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/setting_background"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" &gt;

    &lt;TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="登录界面背景修改："
        android:textColor="#fff"
        android:textSize="30dp" /&gt;

    &lt;LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal" &gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="拍照"
            android:textColor="#fff"
            android:textSize="22dp" /&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="从相册中选取"
            android:textColor="#fff"
            android:textSize="22dp" /&gt;
    &lt;/LinearLayout&gt;

    &lt;TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:text="音效界面背景修改："
        android:textColor="#fff"
        android:textSize="30dp" /&gt;

    &lt;LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal" &gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="拍照"
            android:textColor="#fff"
            android:textSize="22dp" /&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="从相册中选取"
            android:textColor="#fff"
            android:textSize="22dp" /&gt;
    &lt;/LinearLayout&gt;

&lt;/LinearLayout&gt;
```

 

 

 

 

**setting_password.xml**

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/setting_password"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_margin="10dp"
    android:orientation="vertical" &gt;

    &lt;TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="原密码："
        android:textColor="#fff"
        android:textSize="30dp" /&gt;

    &lt;EditText
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:textColor="#fff"
        android:textSize="30dp" 
        android:password="true"/&gt;

    &lt;TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="新密码"
        android:textColor="#fff"
        android:textSize="30dp" /&gt;

    &lt;EditText
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:textColor="#fff"
        android:textSize="30dp" 
        android:password="true"/&gt;

    &lt;Button
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="确定重置密码"
        android:textColor="#fff"
        android:textSize="30dp" /&gt;

&lt;/LinearLayout&gt;
```

 

 

 

 

**setting_sound.xml**

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/setting_sound"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_margin="2dp"
    android:orientation="vertical" &gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="3dp"
        android:orientation="horizontal" &gt;

        &lt;TextView
            android:layout_width="78dp"
            android:layout_height="wrap_content"
            android:text="Steady:"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/DoSteady"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Do"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/setSteady"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/set"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/stopSteady"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/stop"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="3dp"
        android:orientation="horizontal" &gt;

        &lt;TextView
            android:layout_width="78dp"
            android:layout_height="wrap_content"
            android:text="Glad:"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/DoGlad"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Do"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/setGlad"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/set"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/stopGlad"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/stop"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="3dp"
        android:orientation="horizontal" &gt;

        &lt;TextView
            android:layout_width="78dp"
            android:layout_height="wrap_content"
            android:text="Crazy:"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/DoCrazy"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Do"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/setCrazy"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/set"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/stopCrazy"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/stop"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="3dp"
        android:orientation="horizontal" &gt;

        &lt;TextView
            android:layout_width="78dp"
            android:layout_height="wrap_content"
            android:text="Ripe:"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/DoRipe"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Do"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/setRipe"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/set"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/stopRipe"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/stop"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="3dp"
        android:orientation="horizontal" &gt;

        &lt;TextView
            android:layout_width="78dp"
            android:layout_height="wrap_content"
            android:text="Grief:"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/DoGrief"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Do"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/setGrief"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/set"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/stopGrief"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/stop"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="3dp"
        android:orientation="horizontal" &gt;

        &lt;TextView
            android:layout_width="78dp"
            android:layout_height="wrap_content"
            android:text="Help:"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/DoHelp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Do"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/setHelp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/set"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/stopHelp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/stop"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="3dp"
        android:orientation="horizontal" &gt;

        &lt;TextView
            android:layout_width="78dp"
            android:layout_height="wrap_content"
            android:text="Envy:"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/DoEnvy"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Do"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/setEnvy"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/set"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/stopEnvy"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/stop"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="3dp"
        android:orientation="horizontal" &gt;

        &lt;TextView
            android:layout_width="78dp"
            android:layout_height="wrap_content"
            android:text="Yield:"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/DoYield"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Do"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/setYield"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/set"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;

        &lt;Button
            android:id="@+id/stopYield"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/stop"
            android:textColor="#fff"
            android:textSize="15sp" /&gt;
    &lt;/LinearLayout&gt;

&lt;/LinearLayout&gt;
```

 

 

**最后附上源码：**

 

**代码可能注释写的比较少，但是很多都是比较简单的东西，相信有了一定的UI学习经验的人都比较容易看懂，本人打算长期写博客分享所得，倘若有什么好的建议欢迎评论。  **

 

 

 

 

 

 

  

 

  