#android自定义动画插值器(Interpolator)
# 前言

之前已经讲过动画相关的内容，没有接触过的读者可以看下笔者之前对android动画使用的整理。

>  
  


# 插值器概念

动画插值器可以用来控制动画的变化规律，比如变化速率是先快后慢，还是先慢后快，或者是更细节的其他。

android系统已经提供了一些常用的插值器，基本已经能满足大部分需求，对此有兴趣的可以看下笔者前言中的文章，此处就不徒增篇幅了。

对于具体的业务，有时候系统的插值器也没法满足需求。 对于某些场景，一方面需要对动画速率有较细致的控制，另一方面可能使用较为广泛，需要将此逻辑集成，这时就可以使用自定义的插值器。

# 例子（Demo）

本文的例子实现了“弹窗弹性效果”的插值器。 主要有三个动画：
1. 使用自定义插值器，从上至下的动画。1. 使用自定义插值器，从左至右的动画。1. 不使用自定义插值器，实现与插值器类似的效果。（控制上不如插值器细致）
>  
 个人理解： 自定义插值器一般还是要在“对动画运动规律需要细致控制”的前提下，大多数情况其实直接像Demo中的第三个动画一样就可以满足需求了。 


<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039339120.png" width="30%" height="30%"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039344611.png" width="30%" height="30%"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039347942.png" width="30%" height="30%">

# 代码

### MainActivity.java

```
public class MainActivity extends AppCompatActivity implements View.OnClickListener {<!-- -->

  //ui
  private Button btnAnimOne;
  private Button btnAnimTwo;
  private Button btnAnimThree;
  private TextView tvAnim;
  //data
  ObjectAnimator anim;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initViews();
  }

  private void initViews() {<!-- -->
    tvAnim = findViewById(R.id.tv_anim_main);
    btnAnimOne = findViewById(R.id.btn_anim_one_main);
    btnAnimTwo = findViewById(R.id.btn_anim_two_main);
    btnAnimThree = findViewById(R.id.btn_anim_three_main);

    btnAnimOne.setOnClickListener(this);
    btnAnimTwo.setOnClickListener(this);
    btnAnimThree.setOnClickListener(this);
  }

  private void startAnimOne() {<!-- -->
    if (tvAnim == null || anim != null) {<!-- -->
      return;
    }
    tvAnim.setVisibility(View.INVISIBLE);
    anim = ObjectAnimator.ofFloat(tvAnim, "y", 0, tvAnim.getY());
    anim.setDuration(1000);
    anim.setInterpolator(new ElasticityInterpolator());
    anim.addListener(new AnimatorListenerAdapter() {<!-- -->
      @Override public void onAnimationStart(Animator animation) {<!-- -->
        tvAnim.setVisibility(View.VISIBLE);
      }

      @Override public void onAnimationEnd(Animator animation) {<!-- -->
        anim = null;
      }
    });
    anim.start();
  }

  private void startAnimTwo() {<!-- -->
    if (tvAnim == null || anim != null) {<!-- -->
      return;
    }
    tvAnim.setVisibility(View.INVISIBLE);
    anim = ObjectAnimator.ofFloat(tvAnim, "x", 0, tvAnim.getX());
    anim.setDuration(1000);
    anim.setInterpolator(new ElasticityInterpolator());
    anim.addListener(new AnimatorListenerAdapter() {<!-- -->
      @Override public void onAnimationStart(Animator animation) {<!-- -->
        tvAnim.setVisibility(View.VISIBLE);
      }

      @Override public void onAnimationEnd(Animator animation) {<!-- -->
        anim = null;
      }
    });
    anim.start();
  }

  private void startAnimThree() {<!-- -->
    if (tvAnim == null || anim != null) {<!-- -->
      return;
    }
    tvAnim.setVisibility(View.INVISIBLE);
    float x = tvAnim.getX();
    anim = ObjectAnimator.ofFloat(tvAnim, "x", 0, x * 1.2f, x * 0.9f, x * 1.1f, x * 0.9f, x);
    anim.setDuration(1000);
    anim.addListener(new AnimatorListenerAdapter() {<!-- -->
      @Override public void onAnimationStart(Animator animation) {<!-- -->
        tvAnim.setVisibility(View.VISIBLE);
      }

      @Override public void onAnimationEnd(Animator animation) {<!-- -->
        anim = null;
      }
    });
    anim.start();
  }

  @Override public void onClick(View v) {<!-- -->
    int id = v.getId();
    switch (id) {<!-- -->
      case R.id.btn_anim_one_main:
        startAnimOne();
        break;
      case R.id.btn_anim_two_main:
        startAnimTwo();
        break;
      case R.id.btn_anim_three_main:
        startAnimThree();
        break;
    }
  }
}

```

### ElasticityInterpolator.java

```
public class ElasticityInterpolator extends BaseInterpolator {<!-- -->

  /**
   * @param input 表示动画进度，大小从0到1
   */
  @Override public float getInterpolation(float input) {<!-- -->

    if (input &lt;= 0.5) {<!-- -->
      return input * 2;
    } else if (input &lt;= 0.6f) {<!-- -->
      return 1 + (input - 0.5f) * 2;
    } else if (input &lt;= 0.7) {<!-- -->
      return 1.2f - ((input - 0.6f) * 3);
    } else if (input &lt;= 0.8) {<!-- -->
      return 0.9f + ((input - 0.7f) * 2);
    } else if (input &lt;= 0.9) {<!-- -->
      return 1.1f - ((input - 0.8f) * 2);
    } else if (input &lt;= 1) {<!-- -->
      return input;
    }
    return 0;
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
    android:gravity="center"
    android:orientation="vertical"
    tools:context=".MainActivity"
    &gt;
  &lt;TextView
      android:id="@+id/tv_anim_main"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="提示"
      android:textSize="30sp"
      android:textStyle="bold"
      android:elevation="100dp"
      /&gt;
  &lt;Button
      android:id="@+id/btn_anim_one_main"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="竖向动画"
      android:textSize="30sp"
      /&gt;
  &lt;Button
      android:id="@+id/btn_anim_two_main"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="横向动画"
      android:textSize="30sp"
      /&gt;
  &lt;Button
      android:id="@+id/btn_anim_three_main"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="非插值器实现"
      android:textSize="30sp"
      /&gt;

&lt;/LinearLayout&gt;

```