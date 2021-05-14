#给自定义View添加xml属性
笔者之前已经写过了一些自定义View的文章，在此对其也就不从头说起了，如有兴趣的读者可以看一下笔者的前两篇文章。  

笔者之前的文章中仅仅介绍了**如何使用自定义View**以及**为什么要使用自定义View**等等，但是在实际操作中，我们还是希望自定义View之后，直接能够在xml中就对其进行操作，如下图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910923540.png " alt="这里写图片描述">

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910924411.png " alt="这里写图片描述">

# 那么如何操作呢？主要是三个步骤：

## 1、自定义属性名称

## 2、将属性名称与控件关联

## 3、从第三方命名空间获取到自定义属性名称

主要代码： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910928632.png " alt="这里写图片描述">

## 1、自定义属性名称

首先要在values文件中创建一个xml文件，并且在其中写上你需要的自定义属性的名称以及类型。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910929603.png " alt="这里写图片描述">

atts.xml中代码如下：

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;resources&gt;
    &lt;declare-styleable name="MyTitle"&gt;
        &lt;attr name="textColor" format="color"/&gt;
        &lt;attr name="titleText" format="string"/&gt;
        &lt;attr name="leftText" format="string"/&gt;
        &lt;attr name="rightText" format="string"/&gt;
    &lt;/declare-styleable&gt;
&lt;/resources&gt;

```

# 2、将属性名称与控件关联

此点比较简单，直接看代码： MyView.java

```
package com.example.double2.viewxmltest;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Color;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.widget.LinearLayout;
import android.widget.TextView;

/**
 * 项目名称：ViewXmlTest
 * 创建人：Double2号
 * 创建时间：2016/8/4 10:23
 * 修改备注：
 */
public class MyView extends LinearLayout {

    private int colorText;
    private String textLeft;
    private String textTitle;
    private String textRight;
    private TextView tvLeft;
    private TextView tvTitle;
    private TextView tvRight;

    public MyView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        //从xml的属性中获取到字体颜色与string
        TypedArray ta=context.obtainStyledAttributes(attrs,R.styleable.MyTitle);
        colorText=ta.getColor(R.styleable.MyTitle_textColor,Color.BLACK);
        textLeft=ta.getString(R.styleable.MyTitle_leftText);
        textTitle=ta.getString(R.styleable.MyTitle_titleText);
        textRight=ta.getString(R.styleable.MyTitle_rightText);
        ta.recycle();

        //获取到控件
        //加载布局文件，与setContentView()效果一样
        LayoutInflater.from(context).inflate(R.layout.my_view, this);
        tvLeft=(TextView)findViewById(R.id.tv_left);
        tvTitle=(TextView)findViewById(R.id.tv_title);
        tvRight=(TextView)findViewById(R.id.tv_right);

        //将控件与设置的xml属性关联
        tvLeft.setTextColor(colorText);
        tvLeft.setText(textLeft);
        tvTitle.setTextColor(colorText);
        tvTitle.setText(textTitle);
        tvRight.setTextColor(colorText);
        tvRight.setText(textRight);

    }


}


```

my_view.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:orientation="horizontal"
              android:padding="10dp"
              tools:background="@android:color/holo_blue_dark"&gt;

    &lt;TextView
        android:id="@+id/tv_left"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        tools:text="left"
        tools:textColor="#fff"/&gt;

    &lt;TextView
        android:id="@+id/tv_title"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="center"
        android:textSize="23sp"
        tools:text="title"
        tools:textColor="#fff"/&gt;

    &lt;TextView
        android:id="@+id/tv_right"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        tools:text="right"
        tools:textColor="#fff"/&gt;
&lt;/LinearLayout&gt;

```

# 3、从第三方命名空间获取到自定义属性名称

此处要注意在activity_main.xml要申明第三方命名空间(在android studio中只需要用res-auto,在eclipse中就需要加上完整的包名，如下图) 注：my_view只是使用时的一个名称而已，后方的“ <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910931904.png " alt="这里写图片描述">

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910932925.png " alt="这里写图片描述">

activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:my_view="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    &gt;

    &lt;com.example.double2.viewxmltest.MyView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/holo_blue_dark"
        my_view:leftText="Back"
        my_view:rightText="Go"
        my_view:textColor="#fff"
        my_view:titleText="MyViewTest"
        /&gt;

&lt;/RelativeLayout&gt;


```

最后附上源码：