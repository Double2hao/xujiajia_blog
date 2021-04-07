#android XML动画初步解析（activity界面之间跳转demo）
上一篇文章讲了简单的activity界面之间的跳转，并且使用的是android内置的一些动画，此章就小提一下如何自己写一些动画来进行跳转。

 

按例，还是上一下效果：**（结尾附上源码）**

 

<img alt="" class="has" height="500" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/400.png" width="300"> <img alt="" class="has" height="500" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/401.png" width="300">  <img alt="" class="has" height="500" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/402.png" width="300">

 

要自己写动画，首先要对动画的一些属性有一定了解：

**interpolator****:**被用来修饰动画效果，定义动画的变化率，可以使存在的动画效果accelerated(加速)，decelerated(减速),repeated(重复),bounced(弹跳)等。

**android:duration:**动画的持续时间。

**pivotX和pivotY:**这两个属性控制着View对象的支点位置，围绕着这个支点进行旋转和缩放变换。默认情况下，该支点的位置就是View对象的中心。

**Translate:**（有X和Y）这是属性作为一种增量来控制着View对象从它布局容器的左上角坐标偏移的位置。

**rotate:**这个属性控制View对象围绕它的指点进行2D旋转。

**scale:**（有X和Y）这个属性控制着View对象围绕它的指点进行2D缩放。

**alpha:**它表示View对象的透明度。默认值是1（不透明），0带表完全透明（不可见）。

 

笔者已经极力希望描述的清楚一些了，新手仅仅看解起来可能还会概念理有比较大的问题，在后面demo的代码中希望可以再次给读者一些帮助。

demo还是比较简单的，仅仅实现的是两个activity之间的跳转，主要是在xml的文件上需要读者自己去理解一下，当然笔者demo中可尝试的还是有限的，有兴趣的读者可以自己多钻研一下。

 

贴下代码截图：

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/403.png">

 

MainActivity:

 

```
package com.example.animationchanges;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button button=(Button)findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new Intent(MainActivity.this,OneActivity.class);
                startActivity(intent);
                //设置跳转动画
//                overridePendingTransition(R.anim.scale_in,R.anim.scale_out);
//                overridePendingTransition(R.anim.rotate_in,R.anim.rotate_out);
                overridePendingTransition(R.anim.translate_in,R.anim.translate_out);

            }


        });
    }
}

```

 rotate_in:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;set xmlns:android="http://schemas.android.com/apk/res/android"&gt;
&lt;rotate
    android:duration="@android:integer/config_mediumAnimTime"
    android:fromDegrees="0"
    android:pivotX="50%p"
    android:pivotY="50%p"
    android:toDegrees="360"
    /&gt;
&lt;/set&gt;
```

 rotate_out:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;set xmlns:android="http://schemas.android.com/apk/res/android"&gt;
&lt;rotate
    android:duration="@android:integer/config_mediumAnimTime"
    android:fromDegrees="360"
    android:pivotX="50%p"
    android:pivotY="50%p"
    android:toDegrees="0"
    /&gt;
&lt;/set&gt;
```

 

 

scale_in:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/decelerate_interpolator"&gt;
    &lt;scale
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromXScale="2.0"
        android:fromYScale="2.0"
        android:toXScale="1.0"
        android:toYScale="1.0"
        android:pivotX="50%p"
        android:pivotY="50%p"
        /&gt;
&lt;/set&gt;
```

 scale_out:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/decelerate_interpolator"
    android:zAdjustment="top"&gt;
    &lt;scale
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromXScale="1.0"
        android:fromYScale="1.0"
        android:toXScale=".5"
        android:toYScale=".5"
        android:pivotX="50%p"
        android:pivotY="50%p"/&gt;
    &lt;alpha
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromAlpha="1.0"
        android:toAlpha="0" /&gt;
&lt;/set&gt;
```

 translate_in:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/decelerate_interpolator"&gt;
    &lt;translate
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromYDelta="50%p"
        android:toYDelta="0" /&gt;

&lt;/set&gt;
```

 translate_out:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/decelerate_interpolator"&gt;
    &lt;translate
        android:duration="@android:integer/config_mediumAnimTime"
        android:fromYDelta="0"
        android:toYDelta="50%p" /&gt;
&lt;/set&gt;
```

 

 

 

 

 

 

 

源码地址：

 

 

 