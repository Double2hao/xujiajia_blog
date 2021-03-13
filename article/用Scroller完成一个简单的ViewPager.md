#用Scroller完成一个简单的ViewPager
# 前言

ViewPager是我们常用的控件之一，此篇文章我们用Scroller等知识实现一个简单的ViewPager。

# 效果：

## （源码在文章结尾）

<img src="https://img-blog.csdn.net/20170223154857082?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述" title="">

# 涉及知识点

## onMeasure和onLayout

此点若不了解可以参考郭霖前辈的文章：  

## 事件分发机制

此点可以参考笔者文章：  

## scrollTo和scrollBy

**scrollTo：**以View的初始位置为起点进行移动  **scrollBy：**以View的当前位置为起点进行移动

## TouchSlop

系统可以识别出的被认为是滑动的最小距离。如果大于这个距离则是滑动。

## View.getScrollX()

getScrollX（）获取到的值是屏幕的最左侧在整个空间中所占位置的X值。  打个比方：View是一条6米的绳子，而屏幕只能看到2~4米的绳子。那么getScrollX（）的值就为2。如果屏幕看到的是3~5米的绳子，那么getScrollX（）的值就为3。

## Scroller

Scroller的使用主要为3步：  1、初始化Scroller  2、重写computeScroll（）方法  computeScroll（）是在View的draw的时候调用的，而invalidate会导致View重绘，所以在重写computeScroll（）之后，我们要使用invalidate（）来间接调用它。  过程为：invalidate（）-&gt;draw（）-&gt;computeScroll（）  3、使用startScroll（）开启滑动

# 代码：

MainActivity：

```
package com.example.double2.scrollertest;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {<!-- -->

    private Button btnOne;
    private Button btnTwo;
    private Button btnThree;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btnOne=(Button)findViewById(R.id.btn_one);
        btnTwo=(Button)findViewById(R.id.btn_two);
        btnThree=(Button)findViewById(R.id.btn_three);
        btnOne.setOnClickListener(this);
        btnTwo.setOnClickListener(this);
        btnThree.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.btn_one:
                Toast.makeText(this, "One", Toast.LENGTH_SHORT).show();
                break;
            case R.id.btn_two:
                Toast.makeText(this, "Two", Toast.LENGTH_SHORT).show();
                break;
            case R.id.btn_three:
                Toast.makeText(this, "Three", Toast.LENGTH_SHORT).show();
                break;
        }
    }
}

```

ScrollerLayout：

```
package com.example.double2.scrollertest;

import android.content.Context;
import android.support.v4.view.ViewConfigurationCompat;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewConfiguration;
import android.view.ViewGroup;
import android.widget.Scroller;

/**
 * 项目名称：ScrollerTest
 * 创建人：Double2号
 * 创建时间：2017.2.23 12:02
 * 修改备注：
 */
public class ScrollerLayout extends ViewGroup {<!-- -->

    private Scroller mScroller;
    //用于判断是否是滑动
    private int mTouchSlop;
    //按下时的X坐标
    private float downX;
    //滑动时的X坐标
    private float moveX;
    //上次触发ACTION_MOVE事件时的屏幕坐标
    private float lastMoveX;
    //界面可滚动的左边界
    private int leftBorder;
    //界面可滚动的右边界
    private int rightBorder;


    public ScrollerLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        // 创建Scroller的实例
        mScroller = new Scroller(context);
        ViewConfiguration configuration = ViewConfiguration.get(context);
        // 获取TouchSlop值
        mTouchSlop = ViewConfigurationCompat.getScaledPagingTouchSlop(configuration);

    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int childCount = getChildCount();
        for (int i = 0; i &lt; childCount; i++) {
            View childView = getChildAt(i);
            // 为ScrollerLayout中的每一个子控件测量大小
            measureChild(childView, widthMeasureSpec, heightMeasureSpec);
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        if (changed) {
            int childCount = getChildCount();
            for (int i = 0; i &lt; childCount; i++) {
                View childView = getChildAt(i);
                // 为ScrollerLayout中的每一个子控件在水平方向上进行布局
                childView.layout(i * childView.getMeasuredWidth(), 0, (i + 1) * childView.getMeasuredWidth(), childView.getMeasuredHeight());
            }
            // 初始化左右边界值
            leftBorder = getChildAt(0).getLeft();
            rightBorder = getChildAt(getChildCount() - 1).getRight();
        }
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                downX = ev.getRawX();
                lastMoveX = downX;
                break;
            case MotionEvent.ACTION_MOVE:
                moveX = ev.getRawX();
                float diff = Math.abs(moveX - downX);
                lastMoveX = moveX;
                // 当手指拖动值大于TouchSlop值时，认为应该进行滑动，不进行点击事件
                if (diff &gt; mTouchSlop) {
                    return true;
                }
                break;
        }
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                return true;
            case MotionEvent.ACTION_MOVE:
                moveX = event.getRawX();
                int scrolledX = (int) (lastMoveX - moveX);
                if (getScrollX() + scrolledX &lt; leftBorder) {
                    scrollTo(leftBorder, 0);
                    return true;
                } else if (getScrollX() + getWidth() + scrolledX &gt; rightBorder) {
                    scrollTo(rightBorder - getWidth(), 0);
                    return true;
                }
                scrollBy(scrolledX, 0);
                lastMoveX = moveX;
                break;
            case MotionEvent.ACTION_UP:
                // 当手指抬起时，根据当前的滚动值来判定应该滚动到哪个子控件的界面
                //如果当前X超过一半的宽度，那么就向左移
                //如果当前的X小鱼一半的宽度就向右移
                int targetIndex = (getScrollX() + getWidth() / 2) / getWidth();
                int dx = targetIndex * getWidth() - getScrollX();

                mScroller.startScroll(getScrollX(), 0, dx, 0);// 调用startScroll()方法来初始化滚动数据并刷新界面
                invalidate();//通过invalidate（）间接调用computeScroll()

                break;
        }
        return super.onTouchEvent(event);
    }

    @Override
    public void computeScroll() {
        // computeScroll()是不会自己调用，只能通过invalidate()-&gt;draw()-&gt;computeScroll()来间接调用
        //computeScrollOffset()用来判断是否完成了整个华东
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            invalidate();
        }
    }
}
```

activity_main：

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;com.example.double2.scrollertest.ScrollerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
&gt;
    &lt;Button
        android:id="@+id/btn_one"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:text="One"/&gt;

    &lt;Button
        android:id="@+id/btn_two"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:text="Two"/&gt;

    &lt;Button
        android:id="@+id/btn_three"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:text="Three"/&gt;
&lt;/com.example.double2.scrollertest.ScrollerLayout&gt;

```

## 源码地址：

