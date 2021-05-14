#android图片轮播效果，RollViewPager的简单使用
图片轮播算是我们用的比较多的一个功能，我之前也写过类似的文章（），但是说实话自己写并不是特别方便，而且往往bug会比较多。而在github上有一些大神专门写了viewpager的轮播框架并且开源，供大家学习参考，这篇博客就教大家如何简单地使用开源框架RollViewPager。

 

对RollViewPager有兴趣，或者希望更深入学习的可以直接去github下载源码学习：

 

效果：**(源码在文章结尾)**

<img alt="" class="has" height="450" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911590150.png " width="300">

 

**主要支持的一些功能：**

支持无限循环。 触摸时会暂停播放，直到结束触摸一个延迟周期以后继续播放。 看起来就像这样。指示器可以为点可以为数字还可以自定义，位置也可以变。

 

**主要操作过程：**

1、在gradle中导入包：

 

```
    compile 'com.jude:rollviewpager:1.2.9'
```

效果如图：

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911592631.png ">

 

2、设置播放时间间隔、透明度、指示器（指示器可以是默认原点，数字，也可以自定义图片）

 

3、设置适配器,本demo中是StaticPagerAdapter，这个比较简单，用的比较多，有其他需要的可以看github源码。

主要需要设置图片、图片数量等等。

 

 

MainActivity：

 

```
package com.example.double2.rollviewpagertest;

import android.graphics.Color;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;

import com.jude.rollviewpager.RollPagerView;
import com.jude.rollviewpager.adapter.StaticPagerAdapter;
import com.jude.rollviewpager.hintview.ColorPointHintView;

public class MainActivity extends AppCompatActivity {

    private RollPagerView mRollViewPager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mRollViewPager = (RollPagerView) findViewById(R.id.roll_view_pager);

        //设置播放时间间隔
        mRollViewPager.setPlayDelay(1000);
        //设置透明度
        mRollViewPager.setAnimationDurtion(500);
        //设置适配器
        mRollViewPager.setAdapter(new TestNormalAdapter());

        //设置指示器（顺序依次）
        //自定义指示器图片
        //设置圆点指示器颜色
        //设置文字指示器
        //隐藏指示器
        //mRollViewPager.setHintView(new IconHintView(this, R.drawable.point_focus, R.drawable.point_normal));
        mRollViewPager.setHintView(new ColorPointHintView(this, Color.YELLOW,Color.WHITE));
        //mRollViewPager.setHintView(new TextHintView(this));
        //mRollViewPager.setHintView(null);
    }

    private class TestNormalAdapter extends StaticPagerAdapter {
        private int[] imgs = {
                R.drawable.img1,
                R.drawable.img2,
                R.drawable.img3,
                R.drawable.img4,
        };


        @Override
        public View getView(ViewGroup container, int position) {
            ImageView view = new ImageView(container.getContext());
            view.setImageResource(imgs[position]);
            view.setScaleType(ImageView.ScaleType.CENTER_CROP);
            view.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
            return view;
        }


        @Override
        public int getCount() {
            return imgs.length;
        }
    }

}

```

 

 

activity_main:

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:app="http://schemas.android.com/apk/res-auto"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                tools:context=".MainActivity"&gt;

    &lt;com.jude.rollviewpager.RollPagerView
        android:id="@+id/roll_view_pager"
        android:layout_width="match_parent"
        android:layout_height="180dp"
        app:rollviewpager_play_delay="3000"/&gt;
&lt;/RelativeLayout&gt;

```

 

 

源码：

 

 