#（Android Studio 3.0）Android Profiler内存泄漏检查
# 前提概要

内存泄漏是常见又重要的问题，针对这个问题谷歌在Android Studio 3.0中推出了Android Profiler。笔者此篇文章主要记录一下Android Profiler在内存泄漏方面的使用。

# Android Profiler

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2050.png" alt="这里写图片描述">

Android Profiler在Android Studio左下角，需要在Android Studio 3.0及其以上才会有。如果是Android Studio 3.0并且也未有这个按钮，读者也不用着急，运行一下自己的项目就会出现。当点击MEMORY那一行的时候就能进入内存检查的界面。

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2051.png" alt="这里写图片描述">

接下来笔者通过分析内存泄漏的实例的方式来介绍Android Profiler的使用。

# 实例

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2052.png" alt="这里写图片描述">

主要是三个Activity：MainActivity，ActivityOne，ActivityTwo。 MainActivity：主Activity，用于开启内存泄漏的两个Activity。 ActivityOne：通过handler方式泄漏。 ActivityTwo：通过静态引用方式泄漏。

代码如下： **MainActivity：**

```
public class MainActivity extends AppCompatActivity {

    public static Activity activity;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        Button btnOne=findViewById(R.id.btn_one);
        btnOne.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent=new Intent(MainActivity.this,ActivityOne.class);
                startActivity(intent);
            }
        });
        Button btnTwo=findViewById(R.id.btn_two);
        btnTwo.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent=new Intent(MainActivity.this,ActivityTwo.class);
                startActivity(intent);
            }
        });
    }
}

```

**ActivityOne:**

```
public class ActivityOne extends Activity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {

            }
        },1000000);
    }
}

```

**ActivityTwo:**

```
public class ActivityTwo extends Activity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        MainActivity.activity=this;
    }
}

```

# 内存泄漏分析

### 操作

首先我们打开MainActivity，分别开启ActivityOne和ActivityTwo并退出，回到MainActivity。接着打开Android Profiler。

### 检查内存泄漏对象

首先要点击左上方的“Dump Java heap”按钮。（如果是检查内存泄漏，笔者建议在点击之前先点击垃圾回收按钮，以防可回收的存货对象的混淆） <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2053.png" alt="这里写图片描述">

然后就会显示此刻的JAVA堆中对象以及引用情况，我们可以在Heap Dump的右上角选择对象的排列方式，笔者比较推荐按报名排序，因为一般我们检查的都是自己所写的类的泄漏，而非系统层的。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2054.png" alt="这里写图片描述">

如图，我们很快就发现ActivityOne和ActivityTwo泄漏了。 笔者打开了ActivityOne和ActivityTwo之后，回到MainActivity界面并按下垃圾回收按钮。不泄露的情况应该是只有MainActivity被分配了内存，而ActivityOne和ActivityTwo均存活，说明内存没有被释放，即内存泄漏了。

Heap Dump 右边四列的意思分别如下： Alloc Count：Java堆中的实例个数 Native Size：native层分配的内存大小。 Shallow Size：Java堆中分配实际大小 Retained Size：这个类的所有实例保留的内存总大小（并非实际大小）

>  
 在内存泄漏检查的过程中，笔者也出现过理论上对象应该被回收，却仍保留的情况。一般情况下，如果Shallow Size和Retained Size都非常小并且相等，都可以认为是已经被回收的对象。因为系统已经不认为它会被用到，并且没有给它保留分配的内存。 


# 解决内存泄漏（方法一）

继续我们在Heap Dump界面的操作，以检查ActivityOne为例，我们单击它，发现右侧出现了Instance View，然后单击Instance View的对象。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2055.png" alt="这里写图片描述">

在Instance View中，会显示在ActivityOne中的各种对象，而它下方的Reference则是显示诸多对这个存货的ActivityOne对象的引用。大部分都是系统层面的引用，只有一个格外显眼，就是通过“this”对ActivityOne的引用，点进去我们可以发现是MessageQueue持有了这个引用，有点经验的Android程序员马上可以定位到是Handler的内存泄漏了。

# 解决内存泄漏（方法二）

第一种内存泄漏的检查方法由于有过多的系统引用的混淆，相信并不让人觉得容易上手。这时候相信读者会想尝试第三个录制按钮了。 Record memory allocations： 这个按钮的作用是记录一段时间内的内存分配的内容，点击红色的小圆表示开始录制，点击小正方形是结束录制。（录制时间不建议超过10s，计算内存会很慢） <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2056.png" alt="这里写图片描述">

### 操作

首先重新运行APP，停留在MainActivity界面，然后点击红色小圆按钮开始录制，接着分别打开ActivityOne和ActivityTwo然后退出，回到MainActivity界面，最后点击小正方形结束录制。

### 解决内存泄漏

然后我们仍然是按照包名排列，找到内存泄漏的对象。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2057.png" alt="这里写图片描述">

然后我们选择ActivityOne，再单击Instance View 中的这个对象。我们可以发现，完全能再代码中追踪到这个引用创建的地方。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2058.png" alt="这里写图片描述">

我们双击Call Stack中的第一行，发现可以直接跳转到代码内存泄漏的地方。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2059.png" alt="这里写图片描述">

# 两者优劣

解决内存泄漏方法一： 1、可以用于检查内存泄漏，并不仅仅是查看引用情况。 2、不需要定位引用的创建时间，因为查看的是java堆该时的状态。 3、不可以定位到相关代码。 4、系统引用也会显示，容易混淆。

解决内存泄漏方法二： 1、可以直接定位到相关代码。 2、不会有过多的系统引用混淆。 3、需要定位对象创建的时间，在内存记录的时间内进行操作才会显示。

# 总结

Android Profiler只是解决内存泄漏的一个工具，在一些情况下无法定位到相关代码。比如以下情况：

```
A a=A();
……
c=a;

```

在这种情况下，如果A类泄漏，那么代码只能定位到 “A a=A();”，而下面的间接引用却无法由Android Profiler体现出来，这就需要读者通过阅读源码来自行解决了。

总之也算是孰能生巧，新工具固然能提高我们的效率，但是还是无法替代手动检查的工作。