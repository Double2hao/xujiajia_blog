#android中weight的简单使用
## android中对weight的学习可以说是必须的，如果UI布局仅仅使用dp与sp等等，会让布局显得极度不灵活，毕竟各个手机屏幕大小不同，更别说是还有ipad之类的了，所以也是同做本人近期做的一个小UI来分享一下weight的使用。

  

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039753860.png">       <img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039756051.png">

 

左边是各个屏幕的显示效果，右边是1080*1920屏幕的具体显示效果。可以看到，不管屏幕如何变化，使用weight的布局中总是填充满屏幕的，至于美观效果就不说了，直接上代码。

 

本人使用的android studio，eclipse用户直接复制肯定会有问题，AS用户直接复制修改一下中间的图片便可以用啦。当然，weight布局其实比较简单，看一下代码便会了个大概了，详情可以看我的上一篇博客。

 

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;

    &lt;RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="0.5"
        android:background="#7EB345"&gt;

        &lt;Button
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:background="@android:color/transparent"
            android:drawableLeft="@drawable/left_menu"
            android:paddingLeft="17dp" /&gt;

        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:text="某某科技大学"
            android:textSize="25sp" /&gt;

        &lt;Button
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_alignParentRight="true"
            android:background="@android:color/transparent"
            android:text="登陆"
            android:textColor="#fff"
            android:textSize="20sp" /&gt;

    &lt;/RelativeLayout&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1.5"
        android:background="@drawable/school" /&gt;


    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal"&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="校园新闻" /&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="学术公告" /&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="成绩查询" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal"&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="课表信息" /&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="图书借阅" /&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="饭卡消费" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal"&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="校园地图" /&gt;

        &lt;Button
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="在线咨询" /&gt;

        &lt;Button
            android:id="@+id/neirongbuju"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:drawableTop="@mipmap/ic_launcher"
            android:paddingTop="18dp"
            android:text="查看更多" /&gt;
    &lt;/LinearLayout&gt;

    &lt;/LinearLayout&gt;

```

 

 

本人也为新手，闻过则喜，欢迎交流与批评。

 

 

 