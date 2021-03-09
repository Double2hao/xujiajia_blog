#fresco webp动图只播放一次
# 概述

本文适合类似于以下这些需求：
1. 要求webp动图播放一遍后就暂停1. 要求webp动图播放一遍后就消失1. 要求webp动图播放一遍后做其他逻辑
# 具体实现
1. 在ControllerListener中将Animatable对象转化成AnimatedDrawable21. 通过AnimatedDrawable2获取到webp的总帧数1. 每执行一帧记录一次，当记录的帧数等于总帧数的时候认为动画播放了一遍
# 依赖

```
  implementation 'com.facebook.fresco:fresco:1.12.0'
  implementation 'com.facebook.fresco:animated-drawable:1.12.0'//支持AnimatedDrawable2
  implementation 'com.facebook.fresco:animated-webp:1.12.0'//支持webp动图

```

# 权限

```
&lt;uses-permission android:name="android.permission.INTERNET" /&gt;

```

# 初始化

```
public class MyApplication extends Application {<!-- -->
  @Override public void onCreate() {<!-- -->
    super.onCreate();
    Fresco.initialize(this);
  }
}

```

# 源码

### MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

  //constants
  private static final String URI_TEST_WEBP =
      "https://isparta.github.io/compare-webp/image/gif_webp/webp/1.webp";
  //ui
  private SimpleDraweeView dvMain;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initView();
  }

  private void initView() {<!-- -->
    dvMain = findViewById(R.id.dv_main);

    BaseControllerListener baseControllerListener = new BaseControllerListener&lt;ImageInfo&gt;() {<!-- -->
      @Override
      public void onFinalImageSet(String id, ImageInfo imageInfo, Animatable animatable) {<!-- -->
        if (animatable != null &amp;&amp; AnimatedDrawable2.class.isInstance(animatable)) {<!-- -->
          final AnimatedDrawable2 animatedDrawable2 = (AnimatedDrawable2) animatable;
          final int totalCnt = animatedDrawable2.getFrameCount();
          animatedDrawable2.setAnimationListener(new BaseAnimationListener() {<!-- -->
            private int lastFrame; //防止无限循环 适时退出动画

            @Override
            public void onAnimationFrame(AnimatedDrawable2 drawable, int frameNumber) {<!-- -->

              if (!(lastFrame == 0 &amp;&amp; totalCnt &lt;= 1) &amp;&amp; lastFrame &lt;= frameNumber) {<!-- -->
                lastFrame = frameNumber;
              } else {<!-- -->
                animatedDrawable2.stop();
              }
            }

            @Override
            public void onAnimationStart(AnimatedDrawable2 drawable) {<!-- -->
              lastFrame = -1;
            }

            @Override
            public void onAnimationStop(AnimatedDrawable2 drawable) {<!-- -->

            }
          });
        }
      }

      @Override
      public void onFailure(String id, Throwable throwable) {<!-- -->
        Log.e("test", throwable.getMessage());
      }
    };

    PipelineDraweeControllerBuilder builder = Fresco.newDraweeControllerBuilder()
        .setUri(Uri.parse(URI_TEST_WEBP))
        .setOldController(dvMain.getController())
        .setAutoPlayAnimations(true)
        .setControllerListener(baseControllerListener);
    dvMain.setController(builder.build());
  }
}

```

### activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    tools:context=".MainActivity"
    &gt;

  &lt;com.facebook.drawee.view.SimpleDraweeView
      android:id="@+id/dv_main"
      android:layout_width="300dp"
      android:layout_height="300dp"
      app:actualImageScaleType="fitXY"
      /&gt;

&lt;/LinearLayout&gt;

```
