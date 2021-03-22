#Android 可自由拖动的View
# 概述

近期写过一个可自由拖动的View。 需求是能在屏幕中随意拖动这个View，然后如果View最终停止在屏幕左边就想左边靠边，如果停止在屏幕右边就向右边靠边。

>  
 本文是直接把该控件设置成了一个View。 也可以使用PopupWindow来做。 本人使用到的场景需要根据父布局来隐藏和显示，所以就不适合使用PopupWindow了。 


>  
 注意：这个View适合隐藏了ActionBar的情况，如果没有隐藏ActionBar的话还需要开发者根据自己的情况适配一下。 


# demo

<img src="https://img-blog.csdnimg.cn/20191007115405422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# 源码

### MainActivity

```
	public class MainActivity extends AppCompatActivity {<!-- -->

    @Override
    protected void onCreate(Bundle savedInstanceState) {<!-- -->
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        MoveView moveView=findViewById(R.id.mv_main);
        moveView.setOnClickListener(new View.OnClickListener() {<!-- -->
            @Override
            public void onClick(View v) {<!-- -->
                Toast.makeText(MainActivity.this,"test click",Toast.LENGTH_SHORT).show();
            }
        });
    }

}

```

### activity_main

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;


    &lt;com.example.testprojectandroid.ui.MoveView
        android:id="@+id/mv_main"
        android:layout_width="100dp"
        android:layout_height="100dp"/&gt;
&lt;/LinearLayout&gt;

```

### MoveView

```
public class MoveView extends RelativeLayout {<!-- -->

  //data
  private int touchSlop;

  //X和y代表 view中心点的位置
  private float currX;
  private float currY;
  private float lastX;
  private float lastY;

  private float width;
  private float height;

  //记录每次的最终位置
  private int left;
  private int right;
  private int top;
  private int bottom;

  //data
  private boolean hasMovingLong = false; //判断时候移动很大，如果移动过，即使回到原位，也不会触发点击事件

  private View.OnClickListener clickListener;

  public MoveView(Context context) {<!-- -->
    this(context, null);
  }

  public MoveView(Context context, AttributeSet attrs) {<!-- -->
    this(context, attrs, 0);
  }

  public MoveView(Context context, AttributeSet attrs, int defStyle) {<!-- -->
    super(context, attrs, defStyle);

    init();
  }

  private void init() {<!-- -->
    touchSlop = ViewConfiguration.get(getContext()).getScaledTouchSlop();
    //此处设置内容，测试就直接设置为黑色背景
    this.setBackgroundColor(Color.BLACK);
  }

  /**
   * 保证第一次点击的时候currX和currY已经有值
   */
  @Override
  protected void onLayout(boolean changed, int l, int t, int r, int b) {<!-- -->
    super.onLayout(changed, l, t, r, b);
    currX = (l + r) / 2;
    currY = (t + b) / 2;
  }

  @Override
  public boolean onTouchEvent(MotionEvent event) {<!-- -->
    super.onTouchEvent(event);
    if (this.isEnabled()) {<!-- -->
      switch (event.getAction()) {<!-- -->
        case MotionEvent.ACTION_MOVE:

          if (Math.abs(lastX - event.getRawX()) &lt; touchSlop
              &amp;&amp; Math.abs(lastY - event.getRawY()) &lt; touchSlop) {<!-- -->
            return true;
          } else {<!-- -->
            hasMovingLong = true;
          }

          currX = event.getRawX();
          currY = event.getRawY();

          left = (int) (currX - width / 2);
          top = (int) (currY - height / 2);
          right = (int) (currX + width / 2);
          bottom = (int) (currY + height / 2);

          this.layout(left, top, right, bottom);
          break;
        case MotionEvent.ACTION_DOWN:
          width = getWidth();
          height = getHeight();
          lastX = event.getRawX();
          lastY = event.getRawY();
          break;
        case MotionEvent.ACTION_UP:
          currX = event.getRawX();
          currY = event.getRawY();

          //如果移动距离很小，认为是点击
          if (!hasMovingLong) {<!-- -->
            if (Math.abs(lastX - currX) &lt; width / 2 &amp;&amp; Math.abs(lastY - currY) &lt; height / 2) {<!-- -->
              if (clickListener != null) {<!-- -->
                clickListener.onClick(MoveView.this);
              }
            }
          } else {<!-- -->
            hasMovingLong = false;
            updateViewLocation();
          }

          break;
      }
    }
    return true;
  }

  //更新view的位置
  private void updateViewLocation() {<!-- -->
    float parentWidth = ((View) getParent()).getWidth();

    //如果在屏幕左边
    if (parentWidth / 2 &gt; currX) {<!-- -->
      currX = width / 2;
    } else {<!-- -->
      currX = parentWidth - width / 2;
    }

    left = (int) (currX - width / 2);
    top = (int) (currY - height / 2);
    right = (int) (currX + width / 2);
    bottom = (int) (currY + height / 2);

    this.layout(left, top, right, bottom);
  }

  @Override
  public void setOnClickListener(@Nullable View.OnClickListener l) {<!-- -->
    clickListener = l;
  }
}

```

# 拓展

这个View在实际使用的时候会出现部分拖出屏幕的问题，笔者可以自己思考下如何加上top和bottom的限制，实际也不会太复杂，本文就不过多阐述增加篇幅了。