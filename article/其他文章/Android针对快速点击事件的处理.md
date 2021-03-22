#Android针对快速点击事件的处理
# 概述

当对一个按钮快速点击的时候，往往很可能会多次触发同一个逻辑，在有些场景下，会显得极度不合理。而且，这种情况在性能较低的手机上触发概率更高。 比如，点击一个按钮打开登录页面，如果用户点击过快的话，完全可能会跳出两个登录页面。

个人总结了一下针对这种情况的几种处理方式，如有问题或者其他更好的方式可以评论交流。

# 处理方式

### 第一次点击后，让点击事件失效

这种情况更适合于一些耗时的操作，比如网络操作等。 需要注意的是，一般情况下，在操作结束后，需要把点击事件设置为生效。

```
		final Button btn=new Button(this);
        btn.setOnClickListener(new View.OnClickListener() {<!-- -->
            @Override
            public void onClick(View v) {<!-- -->
                btn.setClickable(false);
            }
        });

```

### 使用参数拦截点击事件

一般是设置一个boolean值来判断时候已经点击过，或者时候正在操作中，如果是的话，那么再次点击只会直接return。

```
    private boolean hasSetFlag=false;
    private int flag;

    public void setFlag(int flag) {<!-- -->
        if(hasSetFlag){<!-- -->
            this.flag=flag;
        }
    }

```

### 设置点击间隔

如果两次点击在点击时间间隔内，就不生效。 需要注意的是，FastClickManager尽量都直接写到点击事件的方法中，而不要写到其他逻辑里。否则如果其他逻辑触发了FastClick的逻辑，有可能会导致点击事件失效。

```
public class FastClickManager {<!-- -->
    private static final int DEFAULT_FAST_CLICK_INTERVAL = 500;
    private static long lastClickTime = 0L;

    public FastClickManager() {<!-- -->
    }

    public static boolean isFastDoubleClick() {<!-- -->
        return isFastDoubleClick(DEFAULT_FAST_CLICK_INTERVAL);
    }

    public static boolean isFastDoubleClick(long intervalMillis) {<!-- -->
        long now = SystemClock.elapsedRealtime();
        if (Math.abs(now - lastClickTime) &lt; intervalMillis) {<!-- -->
            return true;
        } else {<!-- -->
            lastClickTime = now;
            return false;
        }
    }
}

		final Button btn=new Button(this);
        btn.setOnClickListener(new View.OnClickListener() {<!-- -->
            @Override
            public void onClick(View v) {<!-- -->
                if(FastClickManager.isFastDoubleClick()){<!-- -->
                    return;
                }
            }
        });

```