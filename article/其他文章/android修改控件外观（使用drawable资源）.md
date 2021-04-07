#android修改控件外观（使用drawable资源）
使用drawable资源这方面是比较容易的，笔者学习并没有花太多时间，但是却是受益匪浅。

之前笔者在做圆角矩形的button的时候，还特意学了PS，想想当时真是天真，android是完全自带这个功能的嘛。

当然android的强大也不仅于此，我们还可以修改seekbar的背景以及移动时的效果，还可以给textview、button等控件加边框，还可以改变它们的形状等等。

本篇文章就做了一个简单的demo，便于大家理解drawable资源的使用：

 

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2870.png" width="400" height="700" alt="">     <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2871.png" width="400" height="700" alt=""> 

 

 

在drawable中创建了比较多的xml文件，在layout中只有activity_main一个：

 

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2872.png" alt=""> 

 

 

activity_main:



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"&gt;

    &lt;EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textColor="@drawable/my_image" /&gt;

    &lt;EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textColor="@drawable/my_image" /&gt;

    &lt;SeekBar
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:max="100"
        android:progressDrawable="@drawable/my_bar" /&gt;

    &lt;ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/layout_logo" /&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/my_shape1"
        android:text="Hello World!" /&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/my_shape2"
        android:text="Hello World!" /&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/my_shape3"
        android:text="Hello World!" /&gt;
&lt;/LinearLayout&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;selector xmlns:android="http://schemas.android.com/apk/res/android"&gt;
   &lt;!--指定获得焦点时的颜色--&gt;
    &lt;item android:state_focused="true"
        android:color="#f44"/&gt;
    &lt;!--指定失去焦点时的颜色--&gt;
    &lt;item android:state_focused="false"
        android:color="#ccf"/&gt;
&lt;/selector&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;layer-list xmlns:android="http://schemas.android.com/apk/res/android"&gt;
   &lt;!--定义轨道背景--&gt;
    &lt;item
        android:id="@android:id/background"
        android:drawable="@drawable/ic_launcher111" /&gt;
    &lt;!--定义轨道上已完成部分的外观--&gt;
    &lt;item
        android:id="@android:id/progress"
        android:drawable="@drawable/ic_launcher222" /&gt;
&lt;/layer-list&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;layer-list xmlns:android="http://schemas.android.com/apk/res/android"&gt;
    &lt;!--定义三个层叠--&gt;
    &lt;item&gt;
        &lt;bitmap android:gravity="center" android:src="@mipmap/ic_launcher" /&gt;
    &lt;/item&gt;
    &lt;item android:left="25dp" android:top="25dp"&gt;
        &lt;bitmap android:gravity="center" android:src="@mipmap/ic_launcher" /&gt;
    &lt;/item&gt;
    &lt;item android:left="50dp" android:top="50dp" &gt;
        &lt;bitmap android:gravity="center" android:src="@mipmap/ic_launcher" /&gt;
    &lt;/item&gt;
&lt;/layer-list&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle"&gt;
    &lt;!--设置填充颜色--&gt;
    &lt;solid android:color="#fff"/&gt;
    &lt;!--设置四周的内边距--&gt;
    &lt;padding android:left="7dp"
        android:top="7dp"
        android:right="7dp"
        android:bottom="7dp"/&gt;
    &lt;!--设置边框--&gt;
    &lt;stroke android:width="3dip" android:color="#ff0"/&gt;
&lt;/shape&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle"&gt;
    &lt;!--定义填充渐变颜色--&gt;
    &lt;gradient
        android:startColor="#FFFF0000"
        android:endColor="#80FF00FF"
        android:angle="45"/&gt;
    &lt;!--设置内填充--&gt;
    &lt;padding android:left="7dp"
        android:top="7dp"
        android:right="7dp"
        android:bottom="7dp"/&gt;
    &lt;!--设置圆角矩形--&gt;
    &lt;corners android:radius="8dp"/&gt;
&lt;/shape&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval"&gt;
    &lt;!--定义填充渐变颜色--&gt;
    &lt;gradient
        android:startColor="#ff0"
        android:endColor="#00f"
        android:angle="45"/&gt;
    &lt;!--设置内填充--&gt;
    &lt;padding android:left="7dp"
        android:top="7dp"
        android:right="7dp"
        android:bottom="7dp"/&gt;
    &lt;!--设置圆角矩形--&gt;
    &lt;corners android:radius="8dp"/&gt;
&lt;/shape&gt;
```



最后附上android studio源码：

 