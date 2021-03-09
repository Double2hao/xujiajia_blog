#coodinatorlayout第一次点击无效的问题
# 问题描述

coodinatorlayout使用AppBarLayout+滑动布局（使用appbar_scrolling_view_behavior），会存在“滑动后第一次点击无效的问题”。 (这里的滑动布局可能是nestedScrollView，ScrollView，RecyclerView，ListView等)

经查看后，发现在这样的布局中，当滑动布局滑动的时候，有时候滑动布局会一直处于fling状态，只有在第一次点击的时候，fling状态才会停止，也因此，第二次点击才会真的生效。

# 解决思路
1. 在发现appBarLayout处于EXPANDED状态的时候，给滑动布局发布一个假的点击事件用于终止fling。1. 使用官方的解决方式，手动判断滑动布局是否滑动结束。
# 制造假的点击事件

```
	getAppBarLayout().addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {<!-- -->

        @Override
        public void onOffsetChanged(final AppBarLayout appBarLayout, final int verticalOffset) {<!-- -->

            if (verticalOffset == 0) {<!-- -->
                // State.EXPANDED
                final long downTime = SystemClock.uptimeMillis();
                final long eventTime = SystemClock.uptimeMillis() + 100;
                final MotionEvent motionEvent = MotionEvent.obtain(downTime, eventTime, MotionEvent.ACTION_UP, 0.0f, 0.0f, 0);
                scorllView.dispatchTouchEvent(motionEvent);
            } else if (Math.abs(verticalOffset) &gt;= appBarLayout.getTotalScrollRange()) {<!-- -->
                // State.COLLAPSED
            } else {<!-- -->
                // State.IDLE
            }
        }
    });

```

# 手动判断滑动是否结束

```
public class FixAppBarLayoutBehavior extends AppBarLayout.Behavior {<!-- -->
 
    public FixAppBarLayoutBehavior() {<!-- -->
        super();
    }
 
    public FixAppBarLayoutBehavior(Context context, AttributeSet attrs) {<!-- -->
        super(context, attrs);
    }
 
    @Override
    public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, AppBarLayout child,
            View target, int dx, int dy, int[] consumed, int type) {<!-- -->
        super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed, type);
        stopNestedScrollIfNeeded(dy, child, target, type);
    }

    @Override
    public void onNestedScroll(CoordinatorLayout coordinatorLayout, AppBarLayout child, View target,
                               int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, int type) {<!-- -->
        super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, type);
        stopNestedScrollIfNeeded(dyUnconsumed, child, target, type);
    }
 
    private void stopNestedScrollIfNeeded(int dy, AppBarLayout child, View target, int type) {<!-- -->
        if (type == ViewCompat.TYPE_NON_TOUCH) {<!-- -->
            final int currOffset = getTopAndBottomOffset();
            if ((dy &lt; 0 &amp;&amp; currOffset == 0) || (dy &gt; 0 &amp;&amp; currOffset == -child.getTotalScrollRange())) {<!-- -->
                ViewCompat.stopNestedScroll(target, ViewCompat.TYPE_NON_TOUCH);
            }
        }
    }
}

```

### xml:

```
	&lt;android.support.design.widget.AppBarLayout
            ...
            app:layout_behavior="yourPackage.FixAppBarLayoutBehavior"&gt;

```

### Java

```
AppBarLayout mAppBarLayout = findViewById(R.id.app_bar);
((CoordinatorLayout.LayoutParams) mAppBarLayout.getLayoutParams()).setBehavior(new FixAppBarLayoutBehavior());

```
