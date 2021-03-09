#onMeasure简单方法 完美解决ListView与ScollView冲突问题！
近期做项目碰到ScrollView与Listview冲突的情况，查看了网上一些解决listview和scollView的冲突的方法，最终选择了**重写onMeasure的方法**来解决这个问题。

在此对各种方法个人做一个总结评价。

 

主要的方法有**四种**：

1、**手动设置ListView高度**（比如把高度设置为200dp）

评价：特别简单无脑，但是大大提高了代码的耦合性，比较适合“图方便”的新手。

 

2、**使用单个ListView的addHeaderView()方法**（给listview设置顶部固定的一个view）

评价：比较简便的方法，但是如果顶部布局需要监听滑动事件，也不可取。 

 

3、**使用LinearLayout取代ListView**（重写LinearLayout）

评价：完全可行，但是让一个LinearLayout来实现Listview的功能真的觉得好奇怪啊。 

 

4、**重写ListView的onMeasure()**

评价：只需要写几行代码，轻松解决冲突问题。不仅降低代码耦合性，而且简单。唯一的缺点，可能就是理解需要花比较多的时间。

 

 

最终效果：（左图为改之前，右图为改之后，**源码在文章结尾**）

<img src="https://img-blog.csdn.net/20160522195648703?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="">  <img src="https://img-blog.csdn.net/20160522195714910?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt=""> 

 

主要实现代码：



```
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE &gt;&gt; 2,
                MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }
```



 

如上所示，**使用expandSpec代替heightMeasureSpec**。很容易理解，就是我们**改变了的listview的高度获取方式**。

那么**MeasureSpec.makeMeasureSpec(int size，int mode)**中的两个参数又是什么呢?

 

**size:**表示父布局提供给你的大小参考

**mode:**表示规格，有EXACTLY、AT_MOST、UNSPECIFIED三种。

 

那么我们代码中填的两个值又分别表示什么呢？

**Integer.MAX_VALUE &gt;&gt; 2：**表示父布局给的参考的大小无限大。（listview无边界） 

**MeasureSpec.AT_MOST：**表示根据布局的大小来确定listview最终的高度，也就是有多少内容就显示多高。 

 

（此处三种方式解释引用郭霖前辈文章中部分内容）



 1. **EXACTLY**

 表示父视图希望子视图的大小应该是由specSize的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

 2. **AT_MOST**

 表示子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

 3. **UNSPECIFIED**

 表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。



倘若读者还有疑问或者对View的绘制过程比较感兴趣，可以参考郭霖前辈的博客：

Android视图绘制流程完全解析，带你一步步深入了解View(二) 



 

 

MainActivity:



```
package com.example.double2.listviewscollview;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.ArrayAdapter;

public class MainActivity extends AppCompatActivity {

    private MyListView mMyListView;
    final private String[] test = {
            "first", "second", "third", "fourth", "fifth",
            "first", "second", "third", "fourth", "fifth"};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initListView();
    }

    private void initListView() {
        mMyListView = (MyListView) findViewById(R.id.lv_main);

        mMyListView.setAdapter(
                new ArrayAdapter&lt;String&gt;(
                        this, android.R.layout.simple_list_item_1, test));
    }
}

```



 



```
package com.example.double2.listviewscollview;

import android.content.Context;
import android.util.AttributeSet;
import android.widget.ListView;

/**
 * 项目名称：ListViewScollView
 * 创建人：Double2号
 * 创建时间：2016/5/22 19:01
 * 修改备注：
 */
public class MyListView extends ListView {

    public MyListView(Context context) {
        super(context);
    }

    public MyListView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyListView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //此处是代码的关键
        //MeasureSpec.AT_MOST的意思就是wrap_content
        //Integer.MAX_VALUE &gt;&gt; 2 是使用最大值的意思,也就表示的无边界模式
        //Integer.MAX_VALUE &gt;&gt; 2 此处表示是福布局能够给他提供的大小
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE &gt;&gt; 2,
                MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }
}

```

** **

**activity_main:**





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        &gt;

        &lt;View
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:background="@android:color/holo_green_light"
            /&gt;

        &lt;com.example.double2.listviewscollview.MyListView
            android:id="@+id/lv_main"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/&gt;
    &lt;/LinearLayout&gt;
&lt;/ScrollView&gt;

```



源码地址：
