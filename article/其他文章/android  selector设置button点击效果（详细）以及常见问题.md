#android  selector设置button点击效果（详细）以及常见问题
button的点击效果学习起来事实上比较容易，此点对开发者来说也是使用的比较频繁的一个知识点，与它相关的还有编辑框的获取焦点时改变背景颜色、选择button选择时改变字体颜色等等。这些其实都是用到的drawable的seletor。

当然drawable中还有很多其他效果可以实现，具体的可以参考笔者的另一篇博客：

# 

 

 

效果：（不点击时显示白色，点击时显示灰色）

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209909865710.png ">

 

实现这个效果其实很简单，在drawable中创建一个xml文件，然后输入两行代码即可解决，如图：

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209909867962.png ">

 

第一行表示点击时显示的图片,第二行表示初始状态显示的图片。

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;selector xmlns:android="http://schemas.android.com/apk/res/android"&gt;
    
    &lt;item android:state_pressed="true" android:drawable="@android:color/darker_gray"/&gt;
    &lt;item android:drawable="@android:color/white"/&gt;

&lt;/selector&gt;
```

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209909867962.png ">

 

然后直接在button的background中设置这个xml文件即可,代码如下：

activity_main:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
&gt;
    &lt;Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="button测试"
        android:background="@drawable/simple_button_style"
        /&gt;
&lt;/LinearLayout&gt;

```

 

 

 

 

 

**常见问题：**

在selector中设置了点击效果和初始状态效果时，点击却没有反应，错误效果以及代码如下:

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209909871603.png ">

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209909872224.png ">

 

 

读者可以发现，与正确的代码相比，后者只是改变了两行代码的顺序。

这里就涉及到seletor选择图片的机制了。**一旦选择到了合适的图片，那么就不会进行之后的判断了。**

 

拿正确的代码举例来说，首先是判断button是否有被点击，如果没有，就不显示灰色，往下继续选择，然后就到了第二行，第二行提供的背景为白色，即显示白色。

在错误的代码中，第一行没有条件，即直接选择白色，跳出选择，就不会进行之后是否有被点击的判断，所以点击效果不会显示。

 

 

如笔者有的不清楚的地方，欢迎读者私信或者评论。对drawable有兴趣的读者可以参考笔者的另一篇博客：