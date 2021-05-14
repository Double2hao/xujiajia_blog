#LinearLayout布局中如何让控件置底
记得刚学习android UI的时候，让控件置底只会使用Relativelayout，有时候会让整体布局很不方便，LinearLayout布局置底的方法很简单，在此只是望和我一样的一些新手，少走些弯路了。

 

效果：

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/760.png">

 

 

代码：

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#d8e0e8"
    android:orientation="vertical" &gt;

    &lt;ListView
        android:id="@+id/msg_list_view"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:divider="#0000" &gt;
    &lt;/ListView&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content" &gt;

        &lt;EditText
            android:id="@+id/input_text"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="Type somthing here"
            android:maxLines="2" /&gt;

        &lt;Button
            android:id="@+id/send"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Send" /&gt;
        
    &lt;/LinearLayout&gt;

&lt;/LinearLayout&gt;
```

 

 

 