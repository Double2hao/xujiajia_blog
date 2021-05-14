#android swipeRefreshLayout 下拉刷新 google官方组件
listview，scrollview，recyclerview等如果碰到更新比较快的数据，没有下拉刷新就显得特别呆板了。于是google就推出了swipeRefreshLayout 这个简单却又实用的组件（知乎APP就是用的这个哦）。

 

笔者博文还是主要给新手提供一个简单的demo学习，效果如下：

 

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911473870.png " width="300" height="500" alt=""> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911475001.png " width="300" height="500" alt=""> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911476552.png " width="300" height="500" alt=""> 

 

代码特别简单，就不多加描述了。

 

MainActivity：



```
package com.example.swiperefreshlayouttest;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.support.v4.widget.SwipeRefreshLayout;
import android.widget.ArrayAdapter;
import android.widget.ListView;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class MainActivity extends Activity implements SwipeRefreshLayout.OnRefreshListener{

    private SwipeRefreshLayout swipeRefreshLayout;
    private ListView listView;
    private ArrayAdapter&lt;String&gt; adapter;
    private List&lt;String&gt; data;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        listView = (ListView) findViewById(R.id.listView);
        swipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.swipeRefreshLayout);

        data = new ArrayList&lt;String&gt;();
        for (int i = 0; i &lt; 20; i++) {
            data.add("当前的item为 " + i);
        }
        adapter = new ArrayAdapter&lt;String&gt;(MainActivity.this, android.R.layout.simple_list_item_1, data);
        listView.setAdapter(adapter);

        //在刷新等待时，滚动条颜色的变化
        swipeRefreshLayout.setColorSchemeResources(android.R.color.holo_blue_bright,
                android.R.color.holo_green_light, android.R.color.holo_orange_light);
        //给swipeRefreshLayout绑定刷新监听
        swipeRefreshLayout.setOnRefreshListener(this);
    }

    @Override
    public void onRefresh() {
        //设置2.5秒的时间来执行以下事件
        new Handler().postDelayed(new Runnable() {
            public void run() {
                data.add(0, "刷新后新增的item");
                adapter.notifyDataSetChanged();
                swipeRefreshLayout.setRefreshing(false);
            }
        }, 2500);
    }
}

```



activity_main:



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;android.support.v4.widget.SwipeRefreshLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swipeRefreshLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;

    &lt;ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:footerDividersEnabled="false" /&gt;

&lt;/android.support.v4.widget.SwipeRefreshLayout&gt;
```



 