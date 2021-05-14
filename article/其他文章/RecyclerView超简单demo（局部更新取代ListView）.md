#RecyclerView超简单demo（局部更新取代ListView）
回顾上一篇写RecyclerView的博客（）。笔者比较具体地讲了一下自己对RecyclerView的理解，但是可能知识点比较杂乱，部分新手读者还是无法领会其意。正好笔者最近学习到“局部更新”的知识点，在此就再附上一个更简单的demo了。

 

为何要实现局部更新？

很简单，就是要提高效率。很多时候在一个列表中，我们只会修改一条或者几条item，如果直接使用notifyDataSetChanged(),未免显得有点浪费资源了。

 

如何实现的局部更新？

在RecyclerView出来之前，ListView如果要实现局部更新，首先要自定义一个ViewHolder，其次要自己写一个局部更新的函数。整体的流程算不上复杂，但是确实不是很方便，开发者们都会不由得想，为什么不直接官方就把这个写好啊。(ListView局部更新的例子在网上有很多，笔者在此处就不多加阐述了。)

然而，RecyclerView的出现就是帮了开发者们一个大忙了，RecyclerView.Adapter自带**局部更新**的函数，不需要自己写，直接调用notifyItemChanged(int position)即可。

 

效果：**（源码在文章结尾）**

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911883250.png " width="300" height="550" alt=""> 

 

使用过程：

1、获取到RecyclerView的Adapter（该Adapter为自己重写的，继承RecyclerView.Adapter）

2、更改数据后，使用Adapter调用notifyItemChanged(int position)。

 

如果读者已经了解了RecyclerView的使用，相信到此就已经理解了。

倘若读者还是刚刚知道RecyclerView，读者可以借鉴此篇文章的demo和之前笔者写的文章学习。

该文章地址： 

 

代码截图：（2个java，2个xml）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911883621.png " alt=""> 

 

MainActivity.java:



```
package com.example.double2.itemupdatarecyclerview;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {

    private Button btnMain;
    private RecyclerView mRecyclerView;
    private MyAdapter mMyAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        mRecyclerView = (RecyclerView) findViewById(R.id.rv_main);
        String[] data = new String[20];
        for (int i = 0; i &lt; 20; i++) {
            data[i] = "item" + i;
        }
        mMyAdapter = new MyAdapter(this, data);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        mRecyclerView.setAdapter(mMyAdapter);

        btnMain = (Button) findViewById(R.id.btn_main);
        btnMain.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mMyAdapter.data[0] = "0000000";
                mMyAdapter.data[2] = "2222222";
                mMyAdapter.notifyItemChanged(0);
                mMyAdapter.notifyItemChanged(2);
            }
        });
    }
}

```



 



```
package com.example.double2.itemupdatarecyclerview;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

/**
 * 项目名称：ItemUpdataRecyclerView
 * 创建人：Double2号
 * 创建时间：2016/6/8 11:00
 * 修改备注：
 */
public class MyAdapter extends RecyclerView.Adapter&lt;MyAdapter.ViewHolder&gt; {

    private Context context;
    public String[] data;

    public MyAdapter(Context context, String[] data){
        this.context=context;
        this.data=data;
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view= LayoutInflater.from(context).inflate(R.layout.item,null);
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        holder.tvItem.setText(data[position]);
    }

    @Override
    public int getItemCount() {
        return data.length;
    }


    public class ViewHolder extends RecyclerView.ViewHolder {
        public TextView tvItem;
        public ViewHolder(View itemView) {
            super(itemView);
            tvItem=(TextView)itemView.findViewById(R.id.tv_item) ;
        }
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
    &gt;

    &lt;Button
        android:id="@+id/btn_main"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="notifyItemChanged"/&gt;

    &lt;android.support.v7.widget.RecyclerView
        android:id="@+id/rv_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        /&gt;
&lt;/LinearLayout&gt;

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;TextView xmlns:android="http://schemas.android.com/apk/res/android"
          android:id="@+id/tv_item"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:padding="10dp"
          android:textSize="20sp"
    /&gt;

```

