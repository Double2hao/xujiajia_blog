#android 图片文字轮播效果（图片和文字自动滚动）
图片轮播是类似知乎日报上的一个轮播效果，如下图。

 

<img alt="" class="has" height="1000" src="https://img-blog.csdn.net/20151004143448452?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="500">

 

 

 

好了直接进入正题，首先是出示一下效果：

 

<img alt="" class="has" height="1000" src="https://img-blog.csdn.net/20151004143503903?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="500">

 

MainActivity：

 

```
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

import android.os.Bundle;
import android.app.Activity;
import android.support.v4.view.PagerAdapter;
import android.support.v4.view.ViewPager;
import android.support.v4.view.ViewPager.OnPageChangeListener;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.ImageView.ScaleType;
import android.widget.LinearLayout;
import android.widget.TextView;

public class MainActivity extends Activity {

    //viewpager
    private ViewPager view_pager;
    private LinearLayout ll_dotGroup;
    private TextView newsTitle;
    private int imgResIds[] = new int[]{R.drawable.a, R.drawable.b,
            R.drawable.c, R.drawable.d, R.drawable.b};
    //存储5张图片
    private String textview[]=new String[]{"12412515125","fawfafawf"
            ,"13f1f12f211","1251f1f12","1t1f12f121"};
    //存储5个目录
    private int curIndex = 0;
    //用来记录当前滚动的位置
    PicsAdapter picsAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        setViewPager();

    }

    private void setViewPager() {

        newsTitle=(TextView)findViewById(R.id.NewsTitle);
        view_pager = (ViewPager) findViewById(R.id.view_pager);
        ll_dotGroup = (LinearLayout) findViewById(R.id.dotgroup);

        picsAdapter = new PicsAdapter(); // 创建适配器
        picsAdapter.setData(imgResIds);
        view_pager.setAdapter(picsAdapter); // 设置适配器

        view_pager.setOnPageChangeListener(new MyPageChangeListener()); //设置页面切换监听器

        initPoints(imgResIds.length); // 初始化图片小圆点
        startAutoScroll(); // 开启自动播放
    }


    // 初始化图片轮播的小圆点和目录
    private void initPoints(int count) {
        for (int i = 0; i &lt; count; i++) {

            ImageView iv = new ImageView(this);
            LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(
                    20, 20);
            params.setMargins(0, 0, 20, 0);
            iv.setLayoutParams(params);

            iv.setImageResource(R.drawable.dot1);

            ll_dotGroup.addView(iv);

        }
        ((ImageView) ll_dotGroup.getChildAt(curIndex))
                .setImageResource(R.drawable.dot2);

        newsTitle.setText(textview[curIndex]);
    }

    // 自动播放
    private void startAutoScroll() {
        ScheduledExecutorService scheduledExecutorService = Executors
                .newSingleThreadScheduledExecutor();
        // 每隔4秒钟切换一张图片
        scheduledExecutorService.scheduleWithFixedDelay(new ViewPagerTask(), 5,
                4, TimeUnit.SECONDS);
    }

    // 切换图片任务
    private class ViewPagerTask implements Runnable {
        @Override
        public void run() {

            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    int count = picsAdapter.getCount();
                    view_pager.setCurrentItem((curIndex + 1) % count);
                }
            });
        }
    }

    // 定义ViewPager控件页面切换监听器
    class MyPageChangeListener implements OnPageChangeListener {

        @Override
        public void onPageScrolled(int position, float positionOffset,
                                   int positionOffsetPixels) {
        }

        @Override
        public void onPageSelected(int position) {
            ImageView imageView1 = (ImageView) ll_dotGroup.getChildAt(position);
            ImageView imageView2 = (ImageView) ll_dotGroup.getChildAt(curIndex);
            if (imageView1 != null) {
                imageView1.setImageResource(R.drawable.dot2);
            }
            if (imageView2 != null) {
                imageView2.setImageResource(R.drawable.dot1);
            }
            curIndex = position;
            newsTitle.setText(textview[curIndex]);

        }


        boolean b = false;

        @Override
        public void onPageScrollStateChanged(int state) {
            //这段代码可不加，主要功能是实现切换到末尾后返回到第一张
            switch (state) {
                case 1:// 手势滑动
                    b = false;
                    break;
                case 2:// 界面切换中
                    b = true;
                    break;
                case 0:// 滑动结束，即切换完毕或者加载完毕
                    // 当前为最后一张，此时从右向左滑，则切换到第一张
                    if (view_pager.getCurrentItem() == view_pager.getAdapter()
                            .getCount() - 1 &amp;&amp; !b) {
                        view_pager.setCurrentItem(0);
                    }
                    // 当前为第一张，此时从左向右滑，则切换到最后一张
                    else if (view_pager.getCurrentItem() == 0 &amp;&amp; !b) {
                        view_pager.setCurrentItem(view_pager.getAdapter()
                                .getCount() - 1);
                    }
                    break;

                default:
                    break;
            }
        }
    }

    // 定义ViewPager控件适配器
    class PicsAdapter extends PagerAdapter {

        private List&lt;ImageView&gt; views = new ArrayList&lt;ImageView&gt;();

        @Override
        public int getCount() {
            if (views == null) {
                return 0;
            }
            return views.size();
        }

        public void setData(int[] imgResIds) {
            for (int i = 0; i &lt; imgResIds.length; i++) {
                ImageView iv = new ImageView(MainActivity.this);
                ViewGroup.LayoutParams params = new ViewGroup.LayoutParams(
                        ViewGroup.LayoutParams.MATCH_PARENT,
                        ViewGroup.LayoutParams.MATCH_PARENT);
                iv.setLayoutParams(params);
                iv.setScaleType(ScaleType.FIT_XY);
                //设置ImageView的属性
                iv.setImageResource(imgResIds[i]);
                views.add(iv);
            }
        }

        public Object getItem(int position) {
            if (position &lt; getCount())
                return views.get(position);
            return null;
        }

        @Override
        public boolean isViewFromObject(View arg0, Object arg1) {
            return arg0 == arg1;
        }

        @Override
        public void destroyItem(View container, int position, Object object) {

            if (position &lt; views.size())
                ((ViewPager) container).removeView(views.get(position));
        }

        @Override
        public int getItemPosition(Object object) {
            return views.indexOf(object);
        }

        @Override
        public Object instantiateItem(View container, int position) {
            if (position &lt; views.size()) {
                final ImageView imageView = views.get(position);
                ((ViewPager) container).addView(imageView);
                return views.get(position);
            }
            return null;
        }

    }

}
```

 

 

 

activity_main:

 

```
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" &gt;

    &lt;RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:layout_marginBottom="5dp"
        android:orientation="vertical" &gt;

        &lt;android.support.v4.view.ViewPager
            android:id="@+id/view_pager"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center" &gt;
        &lt;/android.support.v4.view.ViewPager&gt;

        &lt;RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignParentBottom="true"
            android:background="@drawable/focus_bg"
            &gt;
            &lt;TextView
                android:id="@+id/NewsTitle"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="我校交换生演讲比赛夺冠 美国华盛本大学万里发来贺电"
                android:paddingTop="10dp"
                android:paddingBottom="8dp"
                android:paddingLeft="10dp"
                android:paddingRight="100dp"
                android:textSize="15sp"
                android:textColor="#fff"/&gt;
        &lt;LinearLayout
            android:id="@+id/dotgroup"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:paddingTop="23dp"
            android:paddingRight="10dp"
            android:gravity="center"
            android:layout_marginBottom="15dp"
            android:orientation="horizontal" &gt;
        &lt;/LinearLayout&gt;
    &lt;/RelativeLayout&gt;
    &lt;/RelativeLayout&gt;

&lt;/RelativeLayout&gt;
```

 

 

 

 

 

最后附上源码：

 

 

 

本人博客，android均为新手，闻过则喜，望各位前辈不吝批评指点。
