#android viewpager无限轮播
# 概述

>  
 github地址： 


一直很好奇ViewPager无限轮播的实现方式，于是稍微研究了下，作此文记录之，效果如下： <img src="https://img-blog.csdnimg.cn/20201017105328312.gif" width="30%" height="30%">

# 原理

一般情况下ViewPager的size等于显示的fragment的个数。 无限轮播只需要，将ViewPager的size设置成无限大，通过取余后的值来显示Fragment即可。

>  
 此demo中还加入了timer来自动轮播。并且如果用户手动滑动的话，自动轮播会停止。 


# 关键代码

### 无限轮播

```
public class TestPagerAdapter extends FragmentPagerAdapter {<!-- -->

  //constants
  private final static int CYCLE_COUNT = Integer.MAX_VALUE;

  public TestPagerAdapter(@NonNull FragmentManager fm, int behavior) {<!-- -->
    super(fm, behavior);
  }

  @Override public int getCount() {<!-- -->
    return CYCLE_COUNT;
  }

  @NonNull @Override public Object instantiateItem(@NonNull ViewGroup container, int position) {<!-- -->
    return super.instantiateItem(container, position);
  }

  @NonNull @Override public Fragment getItem(int position) {<!-- -->
    int index = position % 3;
    switch (index) {<!-- -->
      case 0:
        return new TestFragmentOne();
      case 1:
        return new TestFragmentTwo();
      case 2:
        return new TestFragmentThree();
    }
    return new TestFragmentOne();
  }
}

```

### 自动轮播

```
  private void initTimer() {<!-- -->
    cycleTimer = new Timer();
    cycleTimer.schedule(new TimerTask() {<!-- -->
      @Override public void run() {<!-- -->
        runOnUiThread(new Runnable() {<!-- -->
          @Override public void run() {<!-- -->
            if (vpMain != null &amp;&amp; !isTouching) {<!-- -->
              vpMain.setCurrentItem(vpMain.getCurrentItem() + 1);
            }
          }
        });
      }
    }, DELAY_CYCLE, DELAY_CYCLE);
  }

```

### 手动滑动，轮播停止

```
    vpMain.setOnTouchListener(new View.OnTouchListener() {<!-- -->
      @Override public boolean onTouch(View v, MotionEvent event) {<!-- -->
        switch (event.getAction()) {<!-- -->
          case MotionEvent.ACTION_DOWN:
          case MotionEvent.ACTION_MOVE:
            isTouching = true;
            break;
          case MotionEvent.ACTION_UP:
          case MotionEvent.ACTION_CANCEL:
          default:
            isTouching = false;
            break;
        }
        return false;
      }
    });

```
