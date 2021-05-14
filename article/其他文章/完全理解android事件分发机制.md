#完全理解android事件分发机制
# 前言

之前笔者其实已经写过事件分发机制的文章： 但是，现在看来其实更像是一篇知识概括，多处可能未讲清楚，于是打算重写事件分发，用一篇文章大致讲清楚。 首先，形式上笔者最先思考的是**使用源码**，此者能从原理上讲解分发机制，比起侃侃而谈好得多。但是大量的源码往往会让新手产生畏惧难以理解，于是笔者最终还是打算主要使用**实例log输出**来让读者理解android事件分发。

# 重要函数

笔者此次主要提及最常用的几个函数： （其间区别看源码很容易理解，此处直接给上结果） **onClick**（）：这个函数是是View提供给我们的OnClickListener这个接口中的函数，在这里可以自定义对点击事件的处理逻辑。会在onTouchEvent（）中进行调用。 **onTouch**（）：这个函数是View提供给我们的OnTouchListener这个接口中的函数，在这里面可以自定义对触摸事件的处理逻辑。 **onTouchEvent**（）：这个函数是view内部的触摸事件的处理方式，其间包括获取焦点，调用onClick（）等等。 **dispatchTouchEvent**（）：这个是View的事件分发函数，在ViewGroup中进行重写。在View中其间会调用onTouchEvent（），在ViewGroup中其间会调用onInterceptTouchEvent（）和onTouchEvent（）。 **onInterceptTouchEvent**（）：这个函数是事件拦截函数，是ViewGroup才有的函数。

# 重要函数执行顺序

此处我们通过一个很简单的例子进行说明，示例： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039204930.png" alt="这里写图片描述"> 示例xml代码如下：

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;com.example.double2.dispatchevent.LinearLayoutA
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/ll_a"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/holo_blue_dark"
    android:padding="30dp"
    android:orientation="vertical"
    &gt;

    &lt;com.example.double2.dispatchevent.LinearLayoutB
        android:id="@+id/ll_b"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/white"
        android:padding="30dp"
        android:orientation="vertical"
        &gt;

        &lt;com.example.double2.dispatchevent.LinearLayoutC
            android:id="@+id/v_c"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@android:color/black"
            /&gt;
    &lt;/com.example.double2.dispatchevent.LinearLayoutB&gt;
&lt;/com.example.double2.dispatchevent.LinearLayoutA&gt;




```

由外到里主要是三个LinearLayout，分别为A、B、C。笔者分别在五个关键函数中加上了Log，最终点击一下，输出的值如下：

>  
 01-12 01:28:58.136 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A 01-12 01:28:58.136 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A 01-12 01:28:58.136 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B 01-12 01:28:58.136 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_B 01-12 01:28:58.136 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_C 01-12 01:28:58.136 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_C 01-12 01:28:58.136 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_C 01-12 01:28:58.136 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_C 01-12 01:28:58.203 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A 01-12 01:28:58.203 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A 01-12 01:28:58.203 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B 01-12 01:28:58.203 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_B 01-12 01:28:58.203 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_C 01-12 01:28:58.203 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_C 01-12 01:28:58.203 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_C 01-12 01:28:58.203 2442-2442/com.example.double2.dispatchevent D/XJJ: onClick_C 


如上，我们可以看到五个函数的大致执行顺序如下：
1. dispatchTouchEvent（）1. onInterceptTouchEvent（）1. onTouch（）1. onTouchEvent（）1. onClick（）
好奇的读者肯定会问，为什么事件分发执行了两次呢？ 其实很简单，因为的确有两个分发的事件，一次是“手指按下”的事件，一次是“手指抬起”的事件。我们可以看到，只有在“手指抬起”的时候，才会触发**onClick**（）事件。

此处为了便于大家理解，也附上一张事件分发的图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039205331.png" alt="这里写图片描述">

# 控制事件分发

然而还有一个值得我们在意的事情，就是**onTouch**（）以及**onTouchEvent**（）只有在C中执行，而在B和A的就不执行了。 此处，我们就必须再讲一点了。

>  
 dispatchTouchEvent（）返回值为true的时候，事件会继续向下分发，一旦返回值为false，事件就不再向下分发。 onInterceptTouchEvent（）、onTouch（）、onTouchEvent（）这三个函数，返回值为false的时候，事件会继续向下分发，一旦返回值为true，事件就不再向下分发。 而onClick（）没有返回值 


根据这点我们可以知道，一定是C的**onTouchEvent**（）中返回了true，我们将其更改后再看一下效果。 原来的代码为：

```
@Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d("XJJ","onTouchEvent_C");
        return super.onTouchEvent(event);
    }

```

更改后：

```
@Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d("XJJ","onTouchEvent_C");
        return false;
    }

```

效果：

>  
 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_B 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_C 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_C 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_C 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_C 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_B 01-12 01:24:06.250 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_B 01-12 01:24:06.360 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A 01-12 01:24:06.360 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A 01-12 01:24:06.360 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B 01-12 01:24:06.360 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_B 01-12 01:24:06.360 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_B 01-12 01:24:06.360 2442-2442/com.example.double2.dispatchevent D/XJJ: onClick_B 


这样操作之后，我们可以发现，“手指按下”时，**onTouch**（）以及**onTouchEvent**（）事件就可以传递到B了。但是同时，我们也可以发现，当“手指抬起”时，C的**onTouch**（）以及**onTouchEvent**（）事件就不会执行了。

当然，如果需要**onTouch**（）以及**onTouchEvent**（）事件传递到A，那么只需要将B的**onTouchEvent**（）也返回false即可。（此处就不重复演示了）

那么如果**onTouchEvent**（）返回值设置为true了之后，是不是**onClick**（）事件是不是就不会执行了呢？效果如下：

```
01-12 01:39:21.560 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A
01-12 01:39:21.560 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A
01-12 01:39:21.560 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B
01-12 01:39:21.560 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_B
01-12 01:39:21.560 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_C
01-12 01:39:21.560 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_C
01-12 01:39:21.560 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_C
01-12 01:39:21.560 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_C
01-12 01:39:21.617 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A
01-12 01:39:21.617 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A
01-12 01:39:21.617 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B
01-12 01:39:21.617 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_B
01-12 01:39:21.617 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_C
01-12 01:39:21.617 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_C
01-12 01:39:21.617 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_C

```

**onClick**（）的确是不会执行了，如此我们也尝试一下**onTouch**（）返回值设置为true，效果如下：

>  
 01-12 01:40:41.101 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A 01-12 01:40:41.101 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A 01-12 01:40:41.101 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B 01-12 01:40:41.101 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_B 01-12 01:40:41.101 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_C 01-12 01:40:41.101 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_C 01-12 01:40:41.101 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_C 01-12 01:40:41.173 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A 01-12 01:40:41.173 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A 01-12 01:40:41.173 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B 01-12 01:40:41.173 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_B 01-12 01:40:41.173 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_C 01-12 01:40:41.173 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_C 


如此，我们可知，当**onTouch**（）返回值设置为true的时候，**onTouchEvent**（）的确是不会执行了。

到这里，我们其实只剩下对**onInterceptTouchEvent**（） 的分析了，为何没有**dispatchTouchEvent**（）了呢？ 因为**dispatchTouchEvent**（）是事件分发的函数，对View而言，我们阻止它内部的事件分发是没有什么意义的，而我们要控制ViewGroup的事件分发则是通过**onInterceptTouchEvent**（） 来执行的。

>  
 如此我们便假设一个应用场景，A包括B，B包括C，B为横向滑动，C为竖向滑动，当横向滑动的加速度大于竖向滑动的加速度的时候，我们仅仅让B响应事件，而不把事件传递给C。 


我们可以通过**onInterceptTouchEvent（）** 来实现，仅仅只需将B的**onInterceptTouchEvent**（）返回值设置为true即可，效果如下：

>  
 01-12 01:50:18.513 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A 01-12 01:50:18.513 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A 01-12 01:50:18.513 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B 01-12 01:50:18.513 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_B 01-12 01:50:18.513 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_B 01-12 01:50:18.513 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_B 01-12 01:50:18.621 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_A 01-12 01:50:18.621 2442-2442/com.example.double2.dispatchevent D/XJJ: onInterceptTouchEvent_A 01-12 01:50:18.621 2442-2442/com.example.double2.dispatchevent D/XJJ: dispatchTouchEvent_B 01-12 01:50:18.621 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouch_B 01-12 01:50:18.621 2442-2442/com.example.double2.dispatchevent D/XJJ: onTouchEvent_B 01-12 01:50:18.621 2442-2442/com.example.double2.dispatchevent D/XJJ: onClick_B 


## 源码

最后是附上demo的源码，大家可以自行下载理解。 http://download.csdn.net/detail/double2hao/9735367