#android viewpager+fragment做主界面(超容易理解的demo！)
笔者之前已经写过了关于viewpager的demo，但是之前是通过将Activity转化为view，然后放入viewpager实现的，具体操作中有时候还是不如fragment方便。

之前的viewpager demo的地址：

 

主要实现的功能与之前的一样：**（源码在文章结尾）**



 1、滑动的同时改变标题栏

 2、点击标题栏的同时滑动

 

效果：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039633460.png" width="400" height="700" alt="">  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039633781.png" width="400" height="700" alt=""> 

 

主要实现过程：

1、设置好4个fragment，让他们关联其布局文件。

2、为viewpager设置FragmentPagerAdapter。在其中放入每个item中的fragment以及总共的item的个数。

3、为viewpgaer绑定时间监听器，设置在item变化的时候会调用的方法，也就是在滑动的时候会发生的事件。（在demo中item改变时会改变底部button中的字体的颜色）

4、为底部button实现点击事件，事实上就是在点击之后改变viewpgaer中的item，一行解决。

5、为fragment中的button设置toast的点击效果。

 

**主要文件：**

**<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039634092.png" alt=""> **

 

 

**MainActiviy**

```
package com.example.viewpager_fragment;

import android.graphics.Color;
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentPagerAdapter;
import android.support.v4.view.ViewPager;
import android.view.View;
import android.widget.Button;

public class MainActivity extends FragmentActivity
        implements View.OnClickListener {

    private ViewPager viewPager;
    private Button one;
    private Button two;
    private Button three;
    private Button four;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //初始化ViewPager
        InitViewPager();
        //初始化布局
        InitView();

    }


    private void InitViewPager() {
        //获取ViewPager
        //创建一个FragmentPagerAdapter对象，该对象负责为ViewPager提供多个Fragment
        viewPager = (ViewPager) findViewById(R.id.pager);
        FragmentPagerAdapter pagerAdapter = new FragmentPagerAdapter(
                getSupportFragmentManager()) {

            //获取第position位置的Fragment
            @Override
            public Fragment getItem(int position) {
                Fragment fragment = null;
                switch (position) {
                    case 0:
                        fragment = new OneFragment();
                        break;
                    case 1:
                        fragment = new TwoFragment();
                        break;
                    case 2:
                        fragment = new ThreeFragment();
                        break;
                    case 3:
                        fragment = new FourFragment();
                        break;
                }

                return fragment;
            }

            //该方法的返回值i表明该Adapter总共包括多少个Fragment
            @Override
            public int getCount() {
                return 4;
            }

        };
        //为ViewPager组件设置FragmentPagerAdapter
        viewPager.setAdapter(pagerAdapter);

        //为viewpager组件绑定时间监听器
        viewPager.setOnPageChangeListener(new ViewPager.SimpleOnPageChangeListener() {
            //当ViewPager显示的Fragment发生改变时激发该方法
            @Override
            public void onPageSelected(int position) {
                switch (position) {
                    //如果是点击的第一个button，那么就让第一个button的字体变为蓝色
                    //其他的button的字体的颜色变为黑色
                    case 0:
                        one.setTextColor(Color.BLUE);
                        two.setTextColor(Color.BLACK);
                        three.setTextColor(Color.BLACK);
                        four.setTextColor(Color.BLACK);
                        break;
                    case 1:
                        one.setTextColor(Color.BLACK);
                        two.setTextColor(Color.BLUE);
                        three.setTextColor(Color.BLACK);
                        four.setTextColor(Color.BLACK);
                        break;
                    case 2:
                        one.setTextColor(Color.BLACK);
                        two.setTextColor(Color.BLACK);
                        three.setTextColor(Color.BLUE);
                        four.setTextColor(Color.BLACK);
                        break;
                    case 3:
                        one.setTextColor(Color.BLACK);
                        two.setTextColor(Color.BLACK);
                        three.setTextColor(Color.BLACK);
                        four.setTextColor(Color.BLUE);
                        break;
                }
                super.onPageSelected(position);
            }
        });
    }

    private void InitView() {
        one = (Button) findViewById(R.id.bt_one);
        two = (Button) findViewById(R.id.bt_two);
        three = (Button) findViewById(R.id.bt_three);
        four = (Button) findViewById(R.id.bt_four);

        //设置点击监听
        one.setOnClickListener(this);
        two.setOnClickListener(this);
        three.setOnClickListener(this);
        four.setOnClickListener(this);

        //将button中字体的颜色先按照点击第一个button的效果初始化
        one.setTextColor(Color.BLUE);
        two.setTextColor(Color.BLACK);
        three.setTextColor(Color.BLACK);
        four.setTextColor(Color.BLACK);
    }

    //点击主界面上面的button后，将viewpager中的fragment跳转到对应的item
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.bt_one:
                viewPager.setCurrentItem(0);
                break;
            case R.id.bt_two:
                viewPager.setCurrentItem(1);
                break;
            case R.id.bt_three:
                viewPager.setCurrentItem(2);
                break;
            case R.id.bt_four:
                viewPager.setCurrentItem(3);
                break;
        }
    }


}

```



** **

**OneFragment:（由于4个fragment的布局都一样，在此就只放上第一个了）**

****

```
package com.example.viewpager_fragment;

/**
 * Created by 佳佳 on 2015/12/31.
 */

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.Toast;

public class OneFragment extends Fragment {

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

    }

    @Override
    @Nullable
    public View onCreateView(LayoutInflater inflater,
                             @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View rootView = inflater.inflate(R.layout.one, container, false);//关联布局文件

        //设置在OneFragment中的点击效果
        Button bt_frg_one = (Button) rootView.findViewById(R.id.bt_frg_one);
        bt_frg_one.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(getActivity().getApplicationContext(), "这是第一页按钮的点击效果", Toast.LENGTH_SHORT).show();
            }
        });
        return rootView;
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
    }
}

```



**activity_main:**

****

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"&gt;

    &lt;android.support.v4.view.ViewPager
        android:id="@+id/pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"&gt;&lt;/android.support.v4.view.ViewPager&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"&gt;

        &lt;Button
            android:id="@+id/bt_one"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="one"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/bt_two"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="two"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/bt_three"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="three"
            android:textSize="20sp" /&gt;

        &lt;Button
            android:id="@+id/bt_four"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="four"
            android:textSize="20sp" /&gt;

    &lt;/LinearLayout&gt;
&lt;/RelativeLayout&gt;

```



**one:**

****

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;

    &lt;Button
        android:id="@+id/bt_frg_one"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="one"
        android:textSize="30dp" /&gt;
&lt;/LinearLayout&gt;
```



 

最后附上源码地址：

** **

 

 

 