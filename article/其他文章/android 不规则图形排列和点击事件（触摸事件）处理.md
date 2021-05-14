#android 不规则图形排列和点击事件（触摸事件）处理
# 概述

android偶尔会有排列不规则图形的需求，比如平行四边形，梯形等。 对于这些图形，往往会有点击、动画等需求，需要在View本身的绘制和触摸事件上都做一些单独的处理。 本文以平行四边形为例子，记录下对不规则图形的处理。

# demo

demo主要实现了平行四边形的排列和点击。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911704700.png " width="30%" height="30%"> 主要实现过程：
1. 重写onDraw，在矩形的View区域中只画出平行四边形的内容。（最终显示效果中，多个View是重叠的，下图黄色区域才是真正View的大小） <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911707091.png " width="40%" height="40%">1. 给第二第三个View设置leftMargin，让View重叠，这样平行四边形才能看上去是连续排列的。1. 单独处理点击事件。点击到当前View上时，先判断是否在平行四边形区域内，如果在就认为是自己的事件。如果不在平行四边形区域内，那么就把事件抛出，交给下一层处理。
# 代码

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911709452.png " alt="在这里插入图片描述">

## MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

  //test data
  private static final int TEST_SIZE = 400;
  private static final float TEST_WIDTH_WEIGHT = 0.7f;
  //ui
  private LinearLayout llMain;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initViews();
  }

  private void initViews() {<!-- -->
    llMain = findViewById(R.id.ll_main);

    PolygonView testViewOne =
        new PolygonView(this).setDefaultColor(Color.RED).setWidthWeight(TEST_WIDTH_WEIGHT);
    testViewOne.setLayoutParams(new LinearLayout.LayoutParams(TEST_SIZE, TEST_SIZE));
    testViewOne.setOnClickListener(new View.OnClickListener() {<!-- -->
      @Override public void onClick(View v) {<!-- -->
        Toast.makeText(MainActivity.this,"testViewOne",Toast.LENGTH_LONG).show();
      }
    });
    llMain.addView(testViewOne);

    //设置lp ，并且计算margin
    LinearLayout.LayoutParams lp = new LinearLayout.LayoutParams(TEST_SIZE, TEST_SIZE);
    float marginStart = -((1 - TEST_WIDTH_WEIGHT) * TEST_SIZE);
    lp.setMarginStart((int) marginStart);

    PolygonView testViewTwo =
        new PolygonView(this).setDefaultColor(Color.GRAY).setWidthWeight(TEST_WIDTH_WEIGHT);
    testViewTwo.setLayoutParams(lp);
    testViewTwo.setOnClickListener(new View.OnClickListener() {<!-- -->
      @Override public void onClick(View v) {<!-- -->
        Toast.makeText(MainActivity.this,"testViewTwo",Toast.LENGTH_LONG).show();
      }
    });
    llMain.addView(testViewTwo);

    PolygonView testViewThree =
        new PolygonView(this).setDefaultColor(Color.YELLOW).setWidthWeight(TEST_WIDTH_WEIGHT);
    testViewThree.setLayoutParams(lp);
    testViewThree.setOnClickListener(new View.OnClickListener() {<!-- -->
      @Override public void onClick(View v) {<!-- -->
        Toast.makeText(MainActivity.this,"testViewThree",Toast.LENGTH_LONG).show();
      }
    });
    llMain.addView(testViewThree);
  }
}

```

## PolygonView.java

```
public class PolygonView extends View {<!-- -->

  private float widthWeight = 0.5f;
  private int defaultColor = Color.GREEN;
  private int touchColor = Color.BLACK;

  private boolean isTouching = false;

  public PolygonView(Context context) {<!-- -->
    super(context);
  }

  public PolygonView setWidthWeight(float widthWeight) {<!-- -->
    this.widthWeight = widthWeight;
    return this;
  }

  public PolygonView setDefaultColor(int defaultColor) {<!-- -->
    this.defaultColor = defaultColor;
    return this;
  }

  public PolygonView setTouchColor(int touchColor) {<!-- -->
    this.touchColor = touchColor;
    return this;
  }

  @Override
  protected void onDraw(Canvas canvas) {<!-- -->

    canvas.drawColor(Color.TRANSPARENT);//把整张画布绘制成透明

    Paint paint = new Paint();
    paint.setAntiAlias(true);//去锯齿
    if (isTouching) {<!-- -->//设置paint的颜色
      paint.setColor(touchColor);
    } else {<!-- -->
      paint.setColor(defaultColor);
    }
    paint.setStyle(Paint.Style.FILL);//设置paint的风格

    int width = this.getWidth();//获取到view的宽高
    int height = this.getHeight();

    Path path = new Path();    //通过path绘制平行四边形
    path.moveTo(width * (1 - widthWeight), 0);
    path.lineTo(width, 0);
    path.lineTo(width * widthWeight, height);
    path.lineTo(0, height);
    path.close();
    canvas.drawPath(path, paint);
  }

  @Override
  public boolean onTouchEvent(MotionEvent event) {<!-- -->
    switch (event.getAction()) {<!-- -->
      case MotionEvent.ACTION_DOWN:
        isTouching = true;
        invalidate();
        break;
      case MotionEvent.ACTION_UP:
        isTouching = false;
        invalidate();
        break;
    }
    super.onTouchEvent(event);

    //需要return true才能接收到ACTION_DOWN后面的事件
    return true;
  }

  @Override
  public boolean dispatchTouchEvent(MotionEvent event) {<!-- -->
    //判断是否在平行四边形范围内，如果不在就不分发事件
    //isTouching:一旦触摸事件开始，一直到ACTION_UP,一直会处理事件
    if (!isTouching) {<!-- -->
      float x = event.getX();
      float y = event.getY();
      int width = getWidth();
      int height = getHeight();
      float blankWidth = width * (1 - widthWeight);
      if (x &lt; blankWidth) {<!-- -->
        if (y &lt; x * height / blankWidth) {<!-- -->
          return false;
        }
      } else if (x &gt; width * widthWeight) {<!-- -->
        if (y &gt; (x - width + blankWidth) * height / blankWidth) {<!-- -->
          return false;
        }
      }
    }

    return super.dispatchTouchEvent(event);
  }
}


```