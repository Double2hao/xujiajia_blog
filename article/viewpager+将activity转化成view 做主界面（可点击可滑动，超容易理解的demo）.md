#viewpager+将activity转化成view 做主界面（可点击可滑动，超容易理解的demo）
笔者之前已经做过了一个使用viewpgaer轮播效果的博客，但是viewpager本身也是深受androider的喜爱，如今基本每个app都会用到相关的功能，本篇文章也是讲一下用viewpager做主界面的用法。（笔者对viewpager的学习也是比较曲折，网上各种找不到符合自己功能的代码）

 

笔者之后又学习了viewpager+fragment的使用，地址为：

 

主要功能：**（源码在文章最后）**

1、滑动的同时改变标题栏

2、点击标题栏的同时滑动

 

效果：

<img src="https://img-blog.csdn.net/20151205093058605?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt="">     <img src="https://img-blog.csdn.net/20151205093105441?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt=""> 

 

主要文件：

<img src="https://img-blog.csdn.net/20151205093628671?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="">      <img src="https://img-blog.csdn.net/20151205093709767?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt=""> 

 

MainActivity:



```
import android.app.Activity;
import android.app.LocalActivityManager;
import android.content.Intent;
import android.os.Bundle;
import android.support.v4.view.ViewPager;
import android.support.v4.view.ViewPager.OnPageChangeListener;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.ImageView;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends Activity {

    //控件
    private ImageView img1, img2, img3, img4;
    private ViewPager vp;

    //LocalActivityManager用来获取每个activity的view,放于Adapter中
    //MyViewPageAdapter用来放viewpager的内容
    //OnClickListener设置底部图片的点击事件
    //OnPageChangeListener设置图片的滑动事件
    private LocalActivityManager manager;
    private MyViewPageAdapter viewPageAdapter;
    private OnClickListener clickListener;
    private OnPageChangeListener pageChangeListener;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        manager = new LocalActivityManager(this, true);
        manager.dispatchCreate(savedInstanceState);

        vp = (ViewPager) findViewById(R.id.viewpager);
        InitView();
    }


    private void InitView() {
        // TODO Auto-generated method stub
        img1 = (ImageView) findViewById(R.id.main_img1);
        img2 = (ImageView) findViewById(R.id.main_img2);
        img3 = (ImageView) findViewById(R.id.main_img3);
        img4 = (ImageView) findViewById(R.id.main_img4);
        clickListener = new OnClickListener() {

            @Override
            public void onClick(View v) {
                // TODO Auto-generated method stub

                switch (v.getId()) {
                    case R.id.main_img1:
                        vp.setCurrentItem(0);
                        break;
                    case R.id.main_img2:
                        vp.setCurrentItem(1);
                        break;
                    case R.id.main_img3:
                        vp.setCurrentItem(2);
                        break;
                    case R.id.main_img4:
                        vp.setCurrentItem(3);
                        break;
                }
            }
        };

        img1.setOnClickListener(clickListener);
        img2.setOnClickListener(clickListener);
        img3.setOnClickListener(clickListener);
        img4.setOnClickListener(clickListener);
        InitPager();

    }

    private void InitPager() {
        // TODO Auto-generated method stub
        pageChangeListener = new OnPageChangeListener() {
            @Override
            public void onPageSelected(int arg0) {
                // TODO Auto-generated method stub
                switch (arg0) {
                    case 0:
                        img1.setImageResource(R.drawable.main_icon1_1);
                        img2.setImageResource(R.drawable.main_icon2_2);
                        img3.setImageResource(R.drawable.main_icon3_2);
                        img4.setImageResource(R.drawable.main_icon4_2);
                        break;
                    case 1:
                        img1.setImageResource(R.drawable.main_icon1_2);
                        img2.setImageResource(R.drawable.main_icon2_1);
                        img3.setImageResource(R.drawable.main_icon3_2);
                        img4.setImageResource(R.drawable.main_icon4_2);
                        break;
                    case 2:
                        img1.setImageResource(R.drawable.main_icon1_2);
                        img2.setImageResource(R.drawable.main_icon2_2);
                        img3.setImageResource(R.drawable.main_icon3_1);
                        img4.setImageResource(R.drawable.main_icon4_2);
                        break;
                    case 3:
                        img1.setImageResource(R.drawable.main_icon1_2);
                        img2.setImageResource(R.drawable.main_icon2_2);
                        img3.setImageResource(R.drawable.main_icon3_2);
                        img4.setImageResource(R.drawable.main_icon4_1);
                        break;
                }
            }

            @Override
            public void onPageScrolled(int arg0, float arg1, int arg2) {
                // TODO Auto-generated method stub

            }

            @Override
            public void onPageScrollStateChanged(int arg0) {
                // TODO Auto-generated method stub

            }
        };

        AddActivitiesToViewPager();
        vp.setCurrentItem(0);
        vp.setOnPageChangeListener(pageChangeListener);
    }

    private void AddActivitiesToViewPager() {
        List&lt;View&gt; mViews = new ArrayList&lt;View&gt;();
        Intent intent = new Intent();

        intent.setClass(this, Activity_one.class);
        intent.putExtra("id", 1);
        mViews.add(getView("QualityActivity1", intent));

        intent.setClass(this, Activity_two.class);
        intent.putExtra("id", 2);
        mViews.add(getView("QualityActivity2", intent));

        intent.setClass(this, Activity_three.class);
        intent.putExtra("id", 3);
        mViews.add(getView("QualityActivity3", intent));

        intent.setClass(this, Activity_four.class);
        intent.putExtra("id", 4);
        mViews.add(getView("QualityActivity4", intent));

        viewPageAdapter = new MyViewPageAdapter(mViews);
        vp.setAdapter(viewPageAdapter);

    }

    private View getView(String id, Intent intent) {
        return manager.startActivity(id, intent).getDecorView();

    }

}
```



MyViewPagerAdapter:



```
import android.support.v4.view.PagerAdapter;
import android.view.View;
import android.view.ViewGroup;

import java.io.Serializable;
import java.util.List;

public class MyViewPageAdapter extends PagerAdapter implements Serializable {
    private List&lt;View&gt; views;

    public MyViewPageAdapter(List&lt;View&gt; views) {
        this.views = views;
    }

    @Override
    public int getCount() {
        return views.size();
    }

    @Override
    public boolean isViewFromObject(View arg0, Object arg1) {
        return arg0 == arg1;
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        container.removeView(views.get(position));
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        container.addView(views.get(position), 0);
        return views.get(position);
    }
}
```





```
import android.app.Activity;
import android.os.Bundle;

public class Activity_one extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.layout1);

    }
}

```





```
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#F5F3F2"&gt;

    &lt;android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@+id/titile" /&gt;

    &lt;LinearLayout
        android:id="@+id/titile"
        android:layout_width="match_parent"
        android:layout_height="70dp"
        android:layout_alignParentBottom="true"
        android:background="@drawable/tabbar_bg"
        android:orientation="horizontal"&gt;


        &lt;ImageView
            android:id="@+id/main_img1"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:src="@drawable/main_icon1_1" /&gt;


        &lt;ImageView
            android:id="@+id/main_img2"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:src="@drawable/main_icon2_2" /&gt;


        &lt;ImageView
            android:id="@+id/main_img3"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:src="@drawable/main_icon3_2" /&gt;


        &lt;ImageView
            android:id="@+id/main_img4"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:src="@drawable/main_icon4_2" /&gt;

    &lt;/LinearLayout&gt;

&lt;/RelativeLayout&gt;
```



 



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent"&gt;

    &lt;Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="pager1"
        android:textSize="30sp"/&gt;
&lt;/LinearLayout&gt;
```



 

如果是使用eclipse的新手，可以将源码下载下来后，将图片抠出来。（不过还是建议赶紧换android studio哦）

 

 

 

 
