#android LayoutInflater、setContentView、findviewbyid 区分解析
一、LayoutInflater.inflate(layoutId, root, boolen)中三个参数的意义及作用

（这点可以参考鸿洋前辈博客地址：）



主要知识点其实很少，如下：

若temp为layoutId所代表的布局，inflate的三种方法区分如下：

**View view=LayoutInflater.Inflate(layoutId, null )**只创建temp ,temp.LayoutParams=null,返回temp**View view=LayoutInflater.Inflate(layoutId, root, false )**创建temp，temp.LayoutParams=root.LayoutParams;返回temp**View view=LayoutInflater.Inflate(layoutId, root, true )** 创建temp，然后执行root.addView(temp, params);最后返回root



关于点三条可能难以理解，此处笔者自己写了一个例子：

activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:id="@+id/main_layout"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical"&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Main Text View"/&gt;

&lt;/LinearLayout&gt;
```



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;Button xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button" /&gt;
```

MainActivity.java

```
package com.example.double2.layoutinflatertest;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.LinearLayout;

public class MainActivity extends AppCompatActivity {

    private LinearLayout mainLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mainLayout = (LinearLayout) findViewById(R.id.main_layout);
        LayoutInflater layoutInflater = LayoutInflater.from(this);

        View buttonLayout = layoutInflater.inflate(R.layout.button_layout, null);
        mainLayout.addView(buttonLayout);

//        View buttonLayout = layoutInflater.inflate(R.layout.button_layout, mainLayout,true);
    }
}

```







```
        View buttonLayout = layoutInflater.inflate(R.layout.button_layout, null);
        mainLayout.addView(buttonLayout);

```



当执行如下一句时，布局如下：



```
        View buttonLayout = layoutInflater.inflate(R.layout.button_layout, mainLayout,true);

```

<img src="https://img-blog.csdn.net/20160531145649585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="">







二、又上图可知，两种方式都是可以给activity_main中加入button布局了，但是让人奇怪的是，为什么两者是同一个button，显示的却会有差别呢？第二个问题就自然出来了。



解决办法：我们给button_layout中的button加上一个RelativeLayout，如下：



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"&gt;

    &lt;Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button"/&gt;
&lt;/RelativeLayout&gt;

```



<img src="https://img-blog.csdn.net/20160531145649585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" alt="" style="font-size:14px">



可以发现，**这往往就是我们最想要的效果**，就是这个button本来多大，就给它。这样我们修改这个button的时候才比较方便。

那么是什么导致在第一个问题中，button无法显示原来的大小的呢？



```
        View buttonLayout = layoutInflater.inflate(R.layout.button_layout, null);
        mainLayout.addView(buttonLayout);
```



当我们执行如上代码时，root为null，我们获得的buttonLayout的LayoutParams为null。当我们在viewgroup中使用addView(View child)方法，添加一个没有LayoutParams属性的子view。在viewgroup方法内部，它会调用generateDefaultLayoutParams()生成一个LayoutParams赋值个给子view。



LayoutParams有什么用呢？

**简单来说，当一个view的LayoutParams存在时，该布局的宽高设置才会有效。而一个view需要存在LayoutParams，就必须存在父布局。**

关于LayoutParams，可以参考这篇博客：



```
        View buttonLayout = layoutInflater.inflate(R.layout.button_layout, mainLayout,true);
```





三、那么activity_main中的LinearLayout也为空，为什么我们经常使用setContentView(R.id.XXX)的时候却不会造成LayoutParams的缺失而导致布局无法准确显示的问题呢？



如此，我们就需要去了解setContentView()和layoutInflater.inflate()的区别了。

setContentView()设置布局的时候，Android会自动在布局文件的最外层再嵌套一个FrameLayout，所以设置的布局是存在LayoutParams的，也因此可以正常显示。



关于问题二和三，如果还有兴趣的读者，可以参考一下郭霖前辈的博客：





四，findviewbyid()和LayoutInflater.Inflate()有什么区别呢？

findViewById()：从setContentView()已经设置的layout中获取到某个控件。

Inflate()：把某个layout从xml文件中实例化成一个对象。