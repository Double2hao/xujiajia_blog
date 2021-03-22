#android 滑动删除的listview（自定义view）
本篇文章算是对郭霖前辈的一篇文章的详述：

一方面是笔者自己尝试从demo中理解了一下自定义view，另一方面是笔者希望通过更详细的注释已经解说，能帮助新手更容易地理解自定义view的使用。

郭霖前辈原文地址：

 

首先还是展示一下效果：**（源码在文章结尾）**

<img src="https://img-blog.csdn.net/20160329090156470?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300" height="500" alt=""> 

 

**新手比较难理解的几点：（此处新手不懂可以根据源码来看）**

1、onFling()函数，新手可以暂且认为他就是**设置滑动效果**的函数。



```
package com.example.deleteitemlist;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {
    private MyListView myListView;

    private MyAdapter adapter;

    private List&lt;String&gt; contentList = new ArrayList&lt;String&gt;();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //初始化自定的listview中的信息
        initList();

        myListView = (MyListView) findViewById(R.id.my_list_view);
        //实现onDelete接口
        myListView.setOnDeleteListener(new MyListView.OnDeleteListener() {
            @Override
            public void onDelete(int index) {
                //删除传过来的位置的item，并且刷新adapter
                contentList.remove(index);
                adapter.notifyDataSetChanged();
            }
        });
        adapter = new MyAdapter(this, 0, contentList);
        myListView.setAdapter(adapter);
    }

    private void initList() {
        contentList.add("Content Item 1");
        contentList.add("Content Item 2");
        contentList.add("Content Item 3");
        contentList.add("Content Item 4");
        contentList.add("Content Item 5");
        contentList.add("Content Item 6");
        contentList.add("Content Item 7");
        contentList.add("Content Item 8");
        contentList.add("Content Item 9");
        contentList.add("Content Item 10");
        contentList.add("Content Item 11");
        contentList.add("Content Item 12");
        contentList.add("Content Item 13");
        contentList.add("Content Item 14");
        contentList.add("Content Item 15");
        contentList.add("Content Item 16");
        contentList.add("Content Item 17");
        contentList.add("Content Item 18");
        contentList.add("Content Item 19");
        contentList.add("Content Item 20");
    }
}

```





```
package com.example.deleteitemlist;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.TextView;

import java.util.List;

/**
 * 项目名称：DeleteItemList
 * 类描述：
 * 创建人：佳佳
 * 创建时间：2016/3/28 11:16
 * 修改人：佳佳
 * 修改时间：2016/3/28 11:16
 * 修改备注：
 */
public class MyAdapter extends ArrayAdapter&lt;String&gt; {

    public MyAdapter(Context context, int textViewResourceId, List&lt;String&gt; objects) {
        super(context, textViewResourceId, objects);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        View view;
        if (convertView == null) {
            view = LayoutInflater.from(getContext()).inflate(R.layout.my_list_view_item, null);
        } else {
            view = convertView;
        }
        TextView textView = (TextView) view.findViewById(R.id.text_view);
        textView.setText(getItem(position));
        return view;
    }

}

```





```
package com.example.deleteitemlist;

import android.content.Context;
import android.util.AttributeSet;
import android.view.GestureDetector;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ListView;
import android.widget.RelativeLayout;

/**
 * 项目名称：DeleteItemList
 * 类描述：
 * 创建人：佳佳
 * 创建时间：2016/3/28 11:14
 * 修改人：佳佳
 * 修改时间：2016/3/28 11:14
 * 修改备注：
 */
public class MyListView extends ListView implements View.OnTouchListener,
        GestureDetector.OnGestureListener {

    private GestureDetector gestureDetector;
    //设置OnDeleteListener的接口，在MainActivity使用的时候实现
    private OnDeleteListener listener;

    private View deleteButton;

    private ViewGroup itemLayout;

    private int selectedItem;

    private boolean isDeleteShown;

    public MyListView(Context context, AttributeSet attrs) {
        super(context, attrs);
        gestureDetector = new GestureDetector(getContext(), this);
        setOnTouchListener(this);
    }

    public void setOnDeleteListener(OnDeleteListener l) {
        listener = l;
    }

    @Override
    public boolean onTouch(View v, MotionEvent event) {
        //如果点击这个自定义的listview的时候已经显示了一个deleteButton了
        //那么就让这个deleteButton消失，并且让isDeleteShown表示没有deleteButton显示
        if (isDeleteShown) {
            itemLayout.removeView(deleteButton);
            deleteButton = null;
            isDeleteShown = false;
            return false;
        } else {
            //如果没有显示deleteButton，很可能是第一次开启这个view
            //就使用GestureDetector来检测是否触发了特定的手势动作
            return gestureDetector.onTouchEvent(event);
        }
    }

    @Override
    public boolean onDown(MotionEvent e) {
        //如果deleteButton已经显示，那么通过他的xy的位置来获取它在listview中的位置
        if (!isDeleteShown) {
            selectedItem = pointToPosition((int) e.getX(), (int) e.getY());
        }
        return false;
    }

    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,
                           float velocityY) {
        //如果deleteButton不可见 ，而且X的速度的绝对值大于Y的速度的绝对值的时候（手势横向滑动的时候）
        if (!isDeleteShown &amp;&amp; Math.abs(velocityX) &gt; Math.abs(velocityY)) {
            deleteButton = LayoutInflater.from(getContext()).inflate(
                    R.layout.delete_button, null);
            deleteButton.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    itemLayout.removeView(deleteButton);
                    deleteButton = null;
                    isDeleteShown = false;
                    listener.onDelete(selectedItem);
                }
            });
            //此处itemLayout获取到的是textview的父布局RelativeLayout
            itemLayout = (ViewGroup) getChildAt(selectedItem
                    - getFirstVisiblePosition());
            RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(
                    LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
            params.addRule(RelativeLayout.ALIGN_PARENT_RIGHT);
            params.addRule(RelativeLayout.CENTER_VERTICAL);
            //设置好了deleteButton的点击效果、布局，添加这个button
            itemLayout.addView(deleteButton, params);
            //设置isDeleteShown为true，表示deleteButton已经显示
            isDeleteShown = true;
        }
        return false;
    }

    @Override
    public boolean onSingleTapUp(MotionEvent e) {
        return false;
    }

    @Override
    public void onShowPress(MotionEvent e) {

    }

    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX,
                            float distanceY) {
        return false;
    }

    @Override
    public void onLongPress(MotionEvent e) {
    }

    //设置OnDeleteListener的接口，在MainActivity使用的时候实现
    public interface OnDeleteListener {

        void onDelete(int index);

    }

}

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"&gt;

    &lt;com.example.deleteitemlist.MyListView
        android:id="@+id/my_list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"&gt;&lt;/com.example.deleteitemlist.MyListView&gt;

&lt;/RelativeLayout&gt;

```



 



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;Button xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/delete_button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="delete"&gt;

&lt;/Button&gt;  
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    &gt;

    &lt;TextView
        android:id="@+id/text_view"
        android:layout_width="wrap_content"
        android:layout_height="50dp"
        android:layout_centerVertical="true"
        android:gravity="left|center_vertical"
        android:textColor="#000" /&gt;

&lt;/RelativeLayout&gt;
```



源码地址：