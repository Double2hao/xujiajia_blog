#android 自定义View的几种方式
# 概述

本文主要总结一下笔者实际项目中碰到的自定义View的几种方式，以及优劣。

大概有以下几种方式：
1. 使用时绑定1. 不使用xml，直接java实现1. 一次定义，随处可用
>  
 本文更多的还是个人使用经验的总结，如有错误或者需要补充的地方，欢迎评论交流。 


# Demo

demo中分别使用三种方式实现了类似的布局。除了作者将列出的优劣之外，读者可自行查看代码来进行比较。 实现效果与项目文件如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911819520.png " width="20%" height="20%"><img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911820781.png " width="40%" height="40%">

# 一、使用时绑定

**优势**
1. 页面布局与View的业务逻辑解耦。1. 页面布局使用xml实现，可以预览，在页面较复杂的情况下，可以很大的提交开发者的效率。
**劣势**
1. 此方式实现的自定义View无法直接在xml中调用。
**适合场景** 不适合需要在xml中直接调用的场景，其他大部分场景都可以适用。

### TestViewOne.java

```
public class TestViewOne extends LinearLayout {<!-- -->
  //ui
  private TextView tvTitle;
  private TextView tvMessage;

  public TestViewOne(Context context) {<!-- -->
    this(context, null);
  }

  public TestViewOne(Context context, AttributeSet attrs) {<!-- -->
    this(context, attrs, 0);
  }

  public TestViewOne(Context context, AttributeSet attrs, int defStyleAttr) {<!-- -->
    super(context, attrs, defStyleAttr);
  }

  @Override protected void onFinishInflate() {<!-- -->
    super.onFinishInflate();
    bindView();
  }

  private void bindView() {<!-- -->
    tvTitle = findViewById(R.id.tv_title_view_one);
    tvMessage = findViewById(R.id.tv_message_view_one);
  }

  public void setTitle(String title) {<!-- -->
    tvTitle.setText(title);
  }

  public void setMessage(String message) {<!-- -->
    tvMessage.setText(message);
  }
}

```

### view_test_one.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;com.example.multiviewtest.TestViewOne xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="150dp"
    android:gravity="center_vertical"
    android:orientation="vertical"
    android:padding="20dp"
    &gt;
  &lt;TextView
      android:id="@+id/tv_title_view_one"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:textSize="30sp"
      tools:text="@string/test_view_title"
      /&gt;

  &lt;TextView
      android:id="@+id/tv_message_view_one"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:textSize="20sp"
      tools:text="@string/test_view_message"
      /&gt;

&lt;/com.example.multiviewtest.TestViewOne&gt;

```

# 二、不使用xml，直接java实现

**优势**
1. 不需要单独实现xml，如果页面布局不复杂的情况下，可以说是简化的代码的逻辑。
**劣势**
1. 不用xml实现就无法预览，在页面较复杂的情况下，会降低开发的效率。1. 耦合了页面布局和业务逻辑。
**适合场景** 适用于页面布局不复杂的情况。 也适用于给原生的控件增加拓展的场景，比如想给原生的LinearLayout增加header。

### TestViewTwo.java

```
public class TestViewTwo extends LinearLayout {<!-- -->

  //data
  private Context context;
  //ui
  private TextView tvTitle;
  private TextView tvMessage;

  public TestViewTwo(Context context) {<!-- -->
    this(context, null);
  }

  public TestViewTwo(Context context, AttributeSet attrs) {<!-- -->
    this(context, attrs, 0);
  }

  public TestViewTwo(Context context, AttributeSet attrs, int defStyleAttr) {<!-- -->
    super(context, attrs, defStyleAttr);

    this.context = context;

    int padding = DisplayUtils.dpToPx(context, 20);//20dp
    setOrientation(VERTICAL);
    setPadding(padding, padding, padding, padding);
    setLayoutParams(new LayoutParams(LayoutParams.MATCH_PARENT, DisplayUtils.dpToPx(context, 150)));

    createChildView();
  }

  private void createChildView() {<!-- -->
    tvTitle = new TextView(this.context);
    tvTitle.setTextSize(30);
    tvTitle.setLayoutParams(
        new LinearLayout.LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT));
    this.addView(tvTitle);

    tvMessage = new TextView(this.context);
    tvMessage.setTextSize(20);
    tvMessage.setLayoutParams(
        new LinearLayout.LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT));
    this.addView(tvMessage);
  }

  public void setTitle(String title) {<!-- -->
    tvTitle.setText(title);
  }

  public void setMessage(String message) {<!-- -->
    tvMessage.setText(message);
  }
}

```

### 三、一次定义，随处可用

**优势**
1. 将大部分页面布局的逻辑 与 View的业务逻辑解耦。1. 页面布局使用xml实现，可以预览，在页面较复杂的情况下，可以很大的提交开发者的效率。
**劣势**
1. 与方案一相比，需要在非xml中实现父布局的内容。还是将一部分页面布局的逻辑与View的业务逻辑耦合了。
**适合场景** 适用于所有场景，尤其是页面布局复杂的场景。 （与方案一相比，这种方案也可以再xml中直接调用）

>  
 这种方案中，xml中的最外层布局一定要使用merge，否则会增加一层毫无意义的嵌套。 将原来最外层的布局改成merge后，会发现原来的布局无法预览了。此时，给merge加上“tools:parentTag”这个参数就可以了。 


### TestViewThree.java

```
public class TestViewThree extends LinearLayout {<!-- -->
  //ui
  private TextView tvTitle;
  private TextView tvMessage;

  public TestViewThree(Context context) {<!-- -->
    this(context, null);
  }

  public TestViewThree(Context context, AttributeSet attrs) {<!-- -->
    this(context, attrs, 0);
  }

  public TestViewThree(Context context, AttributeSet attrs, int defStyleAttr) {<!-- -->
    super(context, attrs, defStyleAttr);

    LayoutInflater.from(context).inflate(R.layout.view_test_three, this, true);

    int padding = DisplayUtils.dpToPx(context, 20);//20dp
    setOrientation(VERTICAL);
    setPadding(padding, padding, padding, padding);
    setLayoutParams(new LayoutParams(LayoutParams.MATCH_PARENT, DisplayUtils.dpToPx(context, 150)));

    initViews();
  }

  private void initViews() {<!-- -->
    tvTitle = findViewById(R.id.tv_title_view_one);
    tvMessage = findViewById(R.id.tv_message_view_one);
  }

  public void setTitle(String title) {<!-- -->
    tvTitle.setText(title);
  }

  public void setMessage(String message) {<!-- -->
    tvMessage.setText(message);
  }
}

```

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="150dp"
    android:gravity="center_vertical"
    android:orientation="vertical"
    android:padding="20dp"
    tools:parentTag="android.widget.LinearLayout"
    &gt;
  &lt;TextView
      android:id="@+id/tv_title_view_one"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:textSize="30sp"
      tools:text="@string/test_view_title"
      /&gt;

  &lt;TextView
      android:id="@+id/tv_message_view_one"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:textSize="20sp"
      tools:text="@string/test_view_message"
      /&gt;

&lt;/merge&gt;

```

# 其他文件代码

### MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

  //ui
  private LinearLayout llRoot;
  private TestViewOne testViewOne;
  private TestViewTwo testViewTwo;
  private TestViewThree testViewThree;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initViews();
  }

  private void initViews() {<!-- -->
    llRoot = findViewById(R.id.ll_root_main);

    testViewTwo = findViewById(R.id.tv_test_two_main);
    testViewTwo.setTitle("test two title");
    testViewTwo.setMessage("test two message");

    testViewThree = findViewById(R.id.tv_test_three_main);
    testViewThree.setTitle("test three title");
    testViewThree.setMessage("test three message");

    testViewOne =
        (TestViewOne) LayoutInflater.from(this).inflate(R.layout.view_test_one, null, false);
    testViewOne.setTitle("test one title");
    testViewOne.setMessage("test one message");
    llRoot.addView(testViewOne);
  }
}

```

### activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/ll_root_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity"
    &gt;
  &lt;com.example.multiviewtest.TestViewTwo
      android:id="@+id/tv_test_two_main"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      /&gt;
  &lt;com.example.multiviewtest.TestViewThree
      android:id="@+id/tv_test_three_main"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      /&gt;

&lt;/LinearLayout&gt;

```

### DisplayUtils.java

由于实现中会存在单位的差异，因此单独定义了这个工具类来负责单位转化。

```
public class DisplayUtils {<!-- -->
  public static int dpToPx(Context context, float pxValue) {<!-- -->
    float scale = context.getResources().getDisplayMetrics().density;
    return (int) (pxValue * scale);
  }
}

```

# 总结

个人最推荐方案三，原因有以下几点：
1. 除非是非常简单的布局，不然都推荐使用xml来做。用xml实现可预览的布局是AS的一个很实用的特性，不管是开发新功能还是bugfix都能极大的提高效率。1. 页面布局可以和业务逻辑解耦。1. 在java文件和xml中都可以直接调用，适用性最广。