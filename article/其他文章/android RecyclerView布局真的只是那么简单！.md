#android RecyclerView布局真的只是那么简单！
如今android N都已经出来了，作为一个android开发者如果还不知道如何使用android5.X的**RecyclerView**未免有点说不过去了。

RecyclerView比ListView更灵活，更强大。因此也会引入一些复杂性，而这些复杂性，恰恰是在新手前进道路上的很大阻碍，而笔者此文也便是希望可以给予读者一些帮助。 

 

 

**RecyclerView是什么？** 

 

笔者个人看法，RecyclerView只是一个对ListView的升级版，这个升级的主要目的是为了让这个view的效率更高，并且使用更加方便。

我们知道，ListView通过使用ViewHolder来提升性能。ViewHolder通过保存item中使用到的控件的引用来减少findViewById的调用，以此使ListView滑动得更加顺畅。**但这种模式在listview中即使不使用也无妨。**

换言之，在ListView中你不考虑复用的问题也可以，只是你牺牲了内存来方便了代码。但是RecyclerView就不允许你这么做了，你使用RecyclerView就意味着你**一定要复用**，而效果上其实和ListView+ViewHolder差不多。 

 

demo效果：**(源码在文章结尾)**

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911485680.png " width="400" height="600" alt=""> 

 

**主要实现功能：**

1、可以动态排版，选择linearlayou和gridlayout

2、可以增减item

3、实现对item点击事件的监听

4、实现点击事件，点击后能够使item中的字体变成红色

 

 

**RecyclerView如何使用？**（新手学习，推荐一边看笔者demo一边看解释）

 

RecyclerView是support-v7包中的新组件**(此处意味着首先要导入v7包)**，是一个强加的滑动组件，与经典的Listview相比，它同样拥有item回收服用的功能，但是RecyclerView已经封装好了ViewHolder，**用户只需要实现自己的ViewHolder就可以了，该组件会自动帮你复用每一个item。**

** **

**使用RecyclerView笔者认为主要有两个步骤：**

1、设置LayoutManager

2、设置和定义Adapter（主要是实现ViewHolder）

 

由于RecyclerView与listview的使用比较类似，此处还是用大家比较熟悉的listview来解释。

**RecyclerView与listview的使用，主要不同在两个地方：** 

1、需要定义LayoutManager（这点比较简单，不多讲解）

2、listview定义的Adapter主要是针对view来进行操作的，而RecyclerView主要是针对ViewHolder来进行操作的**。**

3、listview本身实现了点击事件，而RecyclerView如果需要点击事件，需要自己写一个接口。（新手不要害怕，并不难）

 

 

下面就分点介绍一下在listview中不会碰到的几个点，也可能是新手认为的RecyclerView的难点：

 

**一、RecyclerView针对ViewHolder来进行操作**

此处主要需要了解RecyclerView必须实现的三个方法:

 

**public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)** 

在任何ViewHolder被实例化的时候，OnCreateViewHolder将会被触发。 

此处实现的内容与fragment中的onCreateView差不多，只是onCreateView最后返回的是view而此处返回的是一个ViewHolder。（注意：使用的时候此处的ViewHolder应该是自己定义的，而不是RecyclerView.ViewHolder）

 

**public void onBindViewHolder(ViewHolder holder, int position)**

此处建立起ViewHolder中视图与数据的关联。由于ViewHolder是自己实现的，此处使用ViewHolder会显得特别自由方便。

 

**public int getItemCount() ** 

这个就不多说了，和listview中的差不多，返回数据的size。

** **

除了这三个方法外，最重要的是需要自己实现一个ViewHolder，这个ViewHolder也需要继承RecyclerView.ViewHolder，（如果需要实现点击事件，也需要应用OnClickListener）

在这个ViewHolder中，可以设置属性，并且与ViewHolder视图内的各个控件绑定，使用起来就十分方便了。

笔者demo中代码：



```
public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
        public TextView tvViewHolder;
        public LinearLayout llViewHolder;

        //初始化viewHolder，此处绑定后在onBindViewHolder中可以直接使用
        public ViewHolder(View itemView){
            super(itemView);
            tvViewHolder=(TextView)itemView.findViewById(R.id.tv_view_holder);
            llViewHolder=(LinearLayout) itemView;
            llViewHolder.setOnClickListener(this);
        }

        //通过接口回调来实现RecyclerView的点击事件
        @Override
        public void onClick(View v) {
            if(mOnItemClickListener!=null) {
                //此处调用的是onItemClick方法，而这个方法是会在RecyclerAdapter被实例化的时候实现
                mOnItemClickListener.onItemClick(v, getItemCount());
            }
        }
    }
```

 

 

二、点击事件需要自己写一个接口

**这个如果花点时间了解概念，其实并不难，主要有以下步骤：（**不懂可以看笔者demo）

 

下面均为笔者demo中的代码；



1、创建一个接口，并在里面写上你需要实现的方法



```
    //定义OnItemClickListener的接口,便于在实例化的时候实现它的点击效果
    public interface OnItemClickListener {
        void onItemClick(View view, int position);
    }
```

2、创建一个该接口的对象来存储监听事件



```
public OnItemClickListener mOnItemClickListener;
```

 

3、在需要使用到该方法的地方进行调用



```
 public ViewHolder(View itemView){
            super(itemView);
            tvViewHolder=(TextView)itemView.findViewById(R.id.tv_view_holder);
            llViewHolder=(LinearLayout) itemView;
            llViewHolder.setOnClickListener(this);
        }
```

 

```
//通过接口回调来实现RecyclerView的点击事件
        @Override
        public void onClick(View v) {
            if(mOnItemClickListener!=null) {
                //此处调用的是onItemClick方法，而这个方法是会在RecyclerAdapter被实例化的时候实现
                mOnItemClickListener.onItemClick(v, getItemCount());
            }
        }
```

 

 

源码截图：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911503941.png " alt=""> 

 



MainActivity:



```
package com.example.double2.recyclerviewtest;

import android.graphics.Color;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.Spinner;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private RecyclerView mRecyclerView;
    private RecyclerAdapter mRecyclerAdapter;
    private RecyclerView.LayoutManager mLayoutManager;
    private Spinner mSpinner;

    private List&lt;String&gt; mData = new ArrayList&lt;String&gt;();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //增加测试数据
        mData.add("Recycler");
        mData.add("Recycler");
        mData.add("Recycler");

        initView();
    }

    private void initView() {
        mRecyclerView = (RecyclerView) findViewById(R.id.rc_main);
        mLayoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(mLayoutManager);
        mRecyclerView.setHasFixedSize(true);

        //设置Spinner
        mSpinner = (Spinner) findViewById(R.id.sp_main);
        List&lt;String&gt; mList = new ArrayList&lt;String&gt;();
        mList.add("LinearLayout");
        mList.add("GridLayout");
        mSpinner.setAdapter(new ArrayAdapter&lt;String&gt;(this, android.R.layout.simple_list_item_1, mList));
        mSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView&lt;?&gt; parent, View view, int position, long id) {
                switch (position) {
                    case 0:
                        //设置为线性布局
                        mRecyclerView.setLayoutManager(new LinearLayoutManager(MainActivity.this));
                        break;
                    case 1:
                        //设置为网格布局,3列
                        mRecyclerView.setLayoutManager(new GridLayoutManager(MainActivity.this, 3));
                        break;
                }
            }

            @Override
            public void onNothingSelected(AdapterView&lt;?&gt; parent) {

            }
        });

        mRecyclerAdapter = new RecyclerAdapter(mData);
        mRecyclerView.setAdapter(mRecyclerAdapter);
        mRecyclerAdapter.setOnItemClickListener(new RecyclerAdapter.OnItemClickListener() {
            //此处实现onItemClick的接口
            @Override
            public void onItemClick(final View view, int position) {
                TextView tvRecycleViewItemText = (TextView) view.findViewById(R.id.tv_view_holder);
                //如果字体本来是黑色就变成红色，反之就变为黑色
                if (tvRecycleViewItemText.getCurrentTextColor() == Color.BLACK)
                    tvRecycleViewItemText.setTextColor(Color.RED);
                else
                    tvRecycleViewItemText.setTextColor(Color.BLACK);
            }
        });

        Button btnAdd = (Button) findViewById(R.id.btn_main_add);
        Button btnDel = (Button) findViewById(R.id.btn_main_del);
        btnAdd.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mData.add("Recycler");
                int position = mData.size();
                if (position &gt; 0)
                    mRecyclerAdapter.notifyDataSetChanged();
            }
        });
        btnDel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int position = mData.size();
                if (position &gt; 0) {
                    mData.remove(position - 1);
                    mRecyclerAdapter.notifyDataSetChanged();
                }
            }
        });

    }
}

```





```
package com.example.double2.recyclerviewtest;

import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.LinearLayout;
import android.widget.TextView;

import java.util.List;

/**
 * 项目名称：RecyclerViewTest
 * 创建人：Double2号
 * 创建时间：2016/4/18 8:12
 * 修改备注：
 */
public class RecyclerAdapter extends RecyclerView.Adapter&lt;RecyclerAdapter.ViewHolder&gt; {
    private List&lt;String&gt; mData;

    public RecyclerAdapter(List&lt;String&gt; data) {
        mData = data;
    }

    //定义一个监听对象，用来存储监听事件
    public OnItemClickListener mOnItemClickListener;

    public void setOnItemClickListener(OnItemClickListener itemClickListener) {
        mOnItemClickListener = itemClickListener;
    }

    //定义OnItemClickListener的接口,便于在实例化的时候实现它的点击效果
    public interface OnItemClickListener {
        void onItemClick(View view, int position);
    }

    public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
        public TextView tvViewHolder;
        public LinearLayout llViewHolder;

        //初始化viewHolder，此处绑定后在onBindViewHolder中可以直接使用
        public ViewHolder(View itemView){
            super(itemView);
            tvViewHolder=(TextView)itemView.findViewById(R.id.tv_view_holder);
            llViewHolder=(LinearLayout) itemView;
            llViewHolder.setOnClickListener(this);
        }

        //通过接口回调来实现RecyclerView的点击事件
        @Override
        public void onClick(View v) {
            if(mOnItemClickListener!=null) {
                //此处调用的是onItemClick方法，而这个方法是会在RecyclerAdapter被实例化的时候实现
                mOnItemClickListener.onItemClick(v, getLayoutPosition());
            }
        }
    }
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View views= LayoutInflater.from(parent.getContext()).inflate(
                R.layout.rc_item,parent,false);
        return new ViewHolder(views);
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        //建立起ViewHolder中试图与数据的关联
         holder.tvViewHolder.setText(mData.get(position)+position);
    }

    @Override
    public int getItemCount() {
        return mData.size();
    }
}

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    &gt;

    &lt;android.support.v7.widget.RecyclerView
        android:id="@+id/rc_main"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"&gt;
    &lt;/android.support.v7.widget.RecyclerView&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"&gt;

        &lt;Spinner
            android:id="@+id/sp_main"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/&gt;

        &lt;Button
            android:id="@+id/btn_main_add"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/add"/&gt;

        &lt;Button
            android:id="@+id/btn_main_del"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/del"/&gt;

    &lt;/LinearLayout&gt;
&lt;/LinearLayout&gt;

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_margin="3dp"
              android:background="@android:color/darker_gray"
              android:gravity="center"
              android:orientation="vertical"
    &gt;

    &lt;ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"/&gt;

    &lt;TextView
        android:id="@+id/tv_view_holder"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@android:color/black"
        android:textSize="20sp"/&gt;
&lt;/LinearLayout&gt;
```



免费源码地址：

 