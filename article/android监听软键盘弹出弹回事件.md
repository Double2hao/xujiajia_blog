#android监听软键盘弹出弹回事件
# 概述

针对“软键盘的遮挡”问题，最简单的方式就是将activity的windowSoftInputMode设置成adjustResize或者adjustPan。

```
android:windowSoftInputMode="adjustResize"
android:windowSoftInputMode="adjustPan"

```

其效果一般如下： <img src="https://img-blog.csdnimg.cn/20210211074645553.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" width="20%" height="20%"> <img src="https://img-blog.csdnimg.cn/20210211074645580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" width="20%" height="20%">

如上图，这样设置后能保证“当前操作的布局”在软键盘之上。 但是实际业务场景，可能需要的是“将整块布局”设置在软键盘之上，即Demo中的两个输入框和按钮都需要在软键盘之上。 如果需要实现这种需求，那么开发者就必须要监听软键盘的弹出弹回事件，然后手动来调整布局。

# 监听源码

监听实现的原理如下：
- 监听当前页面contentView的布局变化- 如果contentView上移，那么是软键盘弹出，如果contentView底部与decorview一样，那么是弹回。
```
public class KeyBoardListenerHelper {<!-- -->
  //constants
  private static final String TAG = "KeyBoardListenerHelper";
  //data
  private WeakReference&lt;Activity&gt; weakActivity = null;//避免内存泄漏，使用弱引用
  private OnKeyBoardChangeListener onKeyBoardChangeListener;
  private final ViewTreeObserver.OnGlobalLayoutListener onGlobalLayoutListener =
      new ViewTreeObserver.OnGlobalLayoutListener() {<!-- -->
        @Override
        public void onGlobalLayout() {<!-- -->
          if (!isActivityValid() || onKeyBoardChangeListener == null) {<!-- -->
            return;
          }
          try {<!-- -->
            Rect rect = new Rect();
            weakActivity.get().getWindow().getDecorView().getWindowVisibleDisplayFrame(rect);
            int screenHeight = weakActivity.get().getWindow().getDecorView().getHeight();
            int keyBoardHeight = screenHeight - rect.bottom;
            onKeyBoardChangeListener.OnKeyBoardChange(keyBoardHeight &gt; 0, keyBoardHeight);
          } catch (Exception e) {<!-- -->
            Log.e(TAG, "onGlobalLayout error:" + e.getMessage());
          }
        }
      };

  public KeyBoardListenerHelper(Activity activity) {<!-- -->
    if (activity == null) {<!-- -->
      return;
    }
    weakActivity = new WeakReference&lt;&gt;(activity);
    try {<!-- -->
      //设置后才可以监听到软键盘的弹出,此处不能设置SOFT_INPUT_ADJUST_UNSPECIFIED或者SOFT_INPUT_STATE_UNSPECIFIED，其他都可以.
      activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE);

      View content = weakActivity.get().findViewById(android.R.id.content);
      content.getViewTreeObserver().addOnGlobalLayoutListener(onGlobalLayoutListener);
    } catch (Exception e) {<!-- -->
      Log.e(TAG, "KeyBoardListenerHelper error:" + e.getMessage());
    }
  }

  //在不使用的时候需要及时销毁，避免内存泄漏或造成额外开销
  public void destroy() {<!-- -->
    Log.i(TAG, "destroy");
    if (!isActivityValid()) {<!-- -->
      return;
    }
    try {<!-- -->
      View content = weakActivity.get().findViewById(android.R.id.content);
      content.getViewTreeObserver().removeOnGlobalLayoutListener(onGlobalLayoutListener);
    } catch (Exception e) {<!-- -->
      Log.e(TAG, "destroy error:" + e.getMessage());
    }
  }

  public void setOnKeyBoardChangeListener(OnKeyBoardChangeListener listener) {<!-- -->
    Log.i(TAG, "setOnKeyBoardChangeListener");
    this.onKeyBoardChangeListener = listener;
  }

  public interface OnKeyBoardChangeListener {<!-- -->

    void OnKeyBoardChange(boolean isShow, int keyBoardHeight);
  }

  public boolean isActivityValid() {<!-- -->
    return weakActivity != null &amp;&amp; weakActivity.get() != null;
  }
}

```

# 监听调用

调用步骤：
1. 在需要的地方初始化1. 根据自己需要在软键盘弹出和弹回的时候控制布局变化1. 在不使用的及时释放不使用的监听器
```
public class MainActivity extends Activity {<!-- -->

  private KeyBoardListenerHelper keyBoardListenerHelper;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    keyBoardListenerHelper = new KeyBoardListenerHelper(this);
    keyBoardListenerHelper.setOnKeyBoardChangeListener(
        new KeyBoardListenerHelper.OnKeyBoardChangeListener() {<!-- -->
          @Override public void OnKeyBoardChange(boolean isShow, int keyBoardHeight) {<!-- -->
            //此处可以根据是否显示软键盘，以及软键盘的高度处理逻辑
            Log.i("testtest", "isShow: " + isShow + " keyBoardHeight:" + keyBoardHeight);
          }
        });
  }

  @Override protected void onDestroy() {<!-- -->
    super.onDestroy();
    if (keyBoardListenerHelper != null) {<!-- -->
      keyBoardListenerHelper.destroy();//避免内存泄漏，需要及时释放
    }
  }
}

```
