#Android实现 曲线路径动画
>  
 本文参考 https://www.jianshu.com/p/fea4d1f6512a 


# 概述

近期碰到曲线动画的实现问题，写本文记录下。

动画类似“剑与远征”游戏的金币动画，动画路径如下图：

# 思路
1. 通过贝塞尔曲线计算出x和y的位置（各个点的位置需要自己微调） （此部分内容参考此文：https://www.jianshu.com/p/fea4d1f6512a）1. 通过ValueAnimator来实现动画
demo如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1360.png" width="40%" height="40%">

>  
 注意： 此demo使用到了屏幕的宽高，因此如果要demo显示正常，需要把状态栏显示透明，把acitionbar去掉。 


# 源码

### MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

  //ui
  private Button btnOne;
  private Button btnTwo;
  private Button btnAnim;
  //data
  private int screenHeight;
  private int screenWidth;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);//设置透明状态栏
    setContentView(R.layout.activity_main);

    screenHeight=getResources().getDisplayMetrics().heightPixels;
    screenWidth=getResources().getDisplayMetrics().widthPixels;
    initViews();
  }

  private void initViews() {<!-- -->
    btnOne = findViewById(R.id.btn_one);
    btnTwo = findViewById(R.id.btn_two);
    btnAnim = findViewById(R.id.btn_anim);

    final ValueAnimator valueAnimator=new ValueAnimator();
    valueAnimator.setDuration(2000);
    valueAnimator.setObjectValues(new PointF(0, 0));
    valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {<!-- -->
      @Override
      public void onAnimationUpdate(ValueAnimator animation) {<!-- -->
        PointF point = (PointF) animation.getAnimatedValue();
        btnAnim.setX(point.x);
        btnAnim.setY(point.y);
      }
    });

    btnOne.setOnClickListener(new View.OnClickListener() {<!-- -->
      @Override public void onClick(View v) {<!-- -->
        valueAnimator.setEvaluator(new TypeEvaluator() {<!-- -->
          @Override public Object evaluate(float fraction, Object startValue, Object endValue) {<!-- -->
            return BezierUtil.calculateBezierPointForQuadratic(fraction,
                new PointF(btnAnim.getLeft(), btnAnim.getTop()),
                new PointF(screenWidth, screenHeight/2),
                new PointF(btnAnim.getLeft(), screenHeight));
          }
        });
        valueAnimator.start();
      }
    });

    btnTwo.setOnClickListener(new View.OnClickListener() {<!-- -->
      @Override public void onClick(View v) {<!-- -->
        valueAnimator.setEvaluator(new TypeEvaluator() {<!-- -->
          @Override public Object evaluate(float fraction, Object startValue, Object endValue) {<!-- -->
            return BezierUtil.calculateBezierPointForCubic(fraction,
                new PointF(btnAnim.getLeft(), btnAnim.getTop()),
                new PointF(screenWidth, screenHeight/3),
                new PointF(0, screenHeight/3*2),
                new PointF(btnAnim.getLeft(), screenHeight));
          }
        });
        valueAnimator.start();
      }
    });
  }
}

```

### BezierUtil.java

```
public class BezierUtil {<!-- -->

  /**
   * B(t) = (1 - t)^2 * P0 + 2t * (1 - t) * P1 + t^2 * P2, t ∈ [0,1]
   *
   * @param t 曲线长度比例
   * @param p0 起始点
   * @param p1 控制点
   * @param p2 终止点
   * @return t对应的点
   */
  public static PointF calculateBezierPointForQuadratic(float t, PointF p0, PointF p1, PointF p2) {<!-- -->
    PointF point = new PointF();
    float temp = 1 - t;
    point.x = temp * temp * p0.x + 2 * t * temp * p1.x + t * t * p2.x;
    point.y = temp * temp * p0.y + 2 * t * temp * p1.y + t * t * p2.y;
    return point;
  }

  /**
   * B(t) = P0 * (1-t)^3 + 3 * P1 * t * (1-t)^2 + 3 * P2 * t^2 * (1-t) + P3 * t^3, t ∈ [0,1]
   *
   * @param t 曲线长度比例
   * @param p0 起始点
   * @param p1 控制点1
   * @param p2 控制点2
   * @param p3 终止点
   * @return t对应的点
   */
  public static PointF calculateBezierPointForCubic(float t, PointF p0, PointF p1, PointF p2,
      PointF p3) {<!-- -->
    PointF point = new PointF();
    float temp = 1 - t;
    point.x = p0.x * temp * temp * temp
        + 3 * p1.x * t * temp * temp
        + 3 * p2.x * t * t * temp
        + p3.x * t * t * t;
    point.y = p0.y * temp * temp * temp
        + 3 * p1.y * t * temp * temp
        + 3 * p2.y * t * t * temp
        + p3.y * t * t * t;
    return point;
  }
}


```

### activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity"
    &gt;

  &lt;Button
      android:id="@+id/btn_one"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:text="二阶贝塞尔曲线"
      /&gt;

  &lt;Button
      android:id="@+id/btn_two"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:text="三阶贝塞尔曲线"
      /&gt;

  &lt;Button
      android:id="@+id/btn_anim"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center"
      android:text="Anim"
      /&gt;
&lt;/LinearLayout&gt;

```