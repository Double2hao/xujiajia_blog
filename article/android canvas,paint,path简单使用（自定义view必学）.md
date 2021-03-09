#android canvas\paint\path简单使用（自定义view必学）
最近学到自定义view，也是涉及到了canvas、paint、path的使用，此内容比较简单。

写此文章，一方面是笔者自己知识点的记录。另一方面希望能给部分新手启发。

 

此三者一般会在自定义view的onDraw()中用到：

**canvas：**决定view的布局（位置，画布颜色，形状）

**paint：**决定view的属性（颜色，字体大小，风格）

**path：**路径（path的用法深入比较复杂，此处由于是入门，就不多加阐述混淆新手了）

 

**大致绘制步骤：**

首先定义paint的属性，然后获取到view的宽高（一般绘制会根据view的大小来适配），随后根据获取到的view的大小使用canvas来绘制出形状。

 

效果：**（代码在文章结尾）**

<img src="https://img-blog.csdn.net/20160401102911831?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300" height="500" alt=""> 

 

MainAcitivity：



```
package com.example.double2.canvaspainttest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

    }
}

```





```
package com.example.double2.canvaspainttest;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Path;
import android.graphics.RectF;
import android.util.AttributeSet;
import android.view.View;

/**
 * Created by Double2号 on 2016/4/1.
 */
public class MyView extends View {


    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //把整张画布绘制成白色
        canvas.drawColor(Color.WHITE);
        Paint paint = new Paint();
        //去锯齿
        //设置paint的颜色
        //设置paint的风格
        //设置画笔宽度
        paint.setAntiAlias(true);
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.STROKE);
        paint.setStrokeWidth(4);
        //获取到view的宽度
        int viewWidth=this.getWidth();
        int viewHight=this.getHeight();
        //绘制圆形
        //drawCircle(float cx, float cy, float radius, Paint paint)
        canvas.drawCircle(viewWidth / 10 + 60, viewHight / 10 + 10, viewWidth / 10 + 10, paint);
        //绘制矩形
        //drawRect(float left, float top, float right, float bottom, Paint paint)
        canvas.drawRect(100, viewHight / 5 + 20, viewWidth / 5 + 10, viewHight * 2 / 5 + 20, paint);

        paint.setStyle(Paint.Style.FILL);
        paint.setColor(Color.RED);
        //绘制圆角矩形
        //RectF(float left, float top, float right, float bottom)
        RectF rel=new RectF(10,viewHight/2 +40,viewWidth/5+10,viewHight*3/5+40);
        canvas.drawRoundRect(rel, 40, 10, paint);

        paint.setTextSize(48);
        //drawText(String text, float x, float y, Paint paint)
        canvas.drawText("xujiajia",viewWidth/2,viewHight/2,paint);

        //通过path绘制一个三角形
        Path path=new Path();
        path.moveTo(300, 400);
        path.lineTo(500, 100);
        path.lineTo(800, 200);
        path.close();
        canvas.drawPath(path, paint);



    }
}

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    &gt;

    &lt;com.example.double2.canvaspainttest.MyView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        /&gt;

&lt;/RelativeLayout&gt;

```



源码地址：
