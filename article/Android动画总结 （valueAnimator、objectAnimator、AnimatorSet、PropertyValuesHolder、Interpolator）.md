#Android动画总结 （valueAnimator、objectAnimator、AnimatorSet、PropertyValuesHolder、Interpolator）
>  
 参考官方文档：https://developer.android.com/guide/topics/graphics/prop-animation#api-overview 


# 概述

笔者近期接触到android动画，将诸多概念都稍微整理了一下。 一方面做一个知识的记录，另一方面也给刚接触android动画的初学者一个参考。

# 主要内容
1. ValueAnimator1. ObjectAnimator（包括AnimatorSet，PropertyValuesHolder）1. AnimatorSet 和 PropertyValuesHolder1. 插值器（Interpolator）
# valueAnimator

valueAnimator可以控制某个值的变化，通过在值变化的同时刷新View来实现动画。 下面的例子是实现TextView中值的刷新：

```
        ValueAnimator valueAnimator = ValueAnimator.ofInt(0, 1000);
        valueAnimator.setDuration(5000);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {<!-- -->
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {<!-- -->
                tv1.setText(animation.getAnimatedValue() + "");
            }
        });
        valueAnimator.start();

```

# ObjectAnimator

ObjectAnimator可以对某个View执行属性动画，常见的属性动画有旋转，平移，缩放等。 ObjectAnimator一般是配合着AnimatorSet或者PropertyValuesHolder使用，demo如下，以下两块代码实现的是同一种效果：“btn先放大后缩小”。

>  
 属性动画有以下几种：（来自官方文档） 
 - translationX,translationY：从当前位置开始平移- rotation, rotationX, rotationY：旋转- scaleX , scaleY：缩放- pivotX , pivotY：旋转和缩放的参照点，默认是View的中心- x , y：平移到当前容器的某个绝对位置- alpha：透明度 


```
		/*
          ObjectAnimator + PropertyValuesHolder
         */
        PropertyValuesHolder xHolder = PropertyValuesHolder.ofFloat("scaleX", 1, 1.5f, 0.8f, 1);
        PropertyValuesHolder yHolder = PropertyValuesHolder.ofFloat("scaleY", 1, 1.5f, 0.8f, 1);
        ObjectAnimator objectAnimator = ObjectAnimator.ofPropertyValuesHolder(btn0, xHolder, yHolder);
        objectAnimator.setDuration(3000);
        objectAnimator.start();

        /*
         * ObjectAnimator + AnimatorSet
         */
        ObjectAnimator anim1 = ObjectAnimator.ofFloat(btn1, "scaleX", 1, 1.5f, 0.8f, 1);
        ObjectAnimator anim2 = ObjectAnimator.ofFloat(btn1, "scaleY", 1, 1.5f, 0.8f, 1);
        final AnimatorSet animatorSet = new AnimatorSet();
        animatorSet.play(anim1);
        animatorSet.play(anim2).with(anim1);
        animatorSet.setDuration(3000);
        animatorSet.start();

```

## AnimatorSet 和 PropertyValuesHolder

### PropertyValuesHolder：

适合在实现“单个View连续动画”的情况下使用，使用AnimatorSet在一些情况下会比PropertyValuesHolder复杂一些。 比如要实现View的循环播放，使用PropertyValuesHolder实现的代码如下：

```
		objectAnimator.setRepeatCount(-1);

```

使用AnimatorSet实现的代码如下：

```
		animatorSet.addListener(new Animator.AnimatorListener() {<!-- -->
            @Override
            public void onAnimationStart(Animator animation) {<!-- -->

            }

            @Override
            public void onAnimationEnd(Animator animation) {<!-- -->
                if (animatorSet != null) {<!-- -->
                    animatorSet.start();
                }
            }

            @Override
            public void onAnimationCancel(Animator animation) {<!-- -->

            }

            @Override
            public void onAnimationRepeat(Animator animation) {<!-- -->

            }
        });

```

### AnimatorSet

适合在动画比较复杂的情况下使用，比如有多个View的动画需要同时进行或者交替进行，这种情况使用PropertyValuesHolder是很难实现的。 再比如，对于同一个View的动画不连续的情况，PropertyValuesHolder也比较难实现，或者说实现更加复杂。 demo中，View执行完平移之后再执行缩放逻辑。

```
		ObjectAnimator anim11 = ObjectAnimator.ofFloat(tv2, "translationX", 200);
        ObjectAnimator anim22 = ObjectAnimator.ofFloat(tv2, "scaleX", 1, 1.5f, 1f);
        ObjectAnimator anim33 = ObjectAnimator.ofFloat(tv2, "scaleY", 1, 1.5f, 1f);
        final AnimatorSet animatorSet2 = new AnimatorSet();
        animatorSet2.play(anim11);
        animatorSet2.play(anim22).after(anim11);
        animatorSet2.play(anim33).with(anim22);
        animatorSet2.setDuration(3000);
        animatorSet2.start();

```

# 插值器（Interpolator）

插值器可以控制动画变化的速率，设置非常简单，代码如下：

```
        ValueAnimator valueAnimator = ValueAnimator.ofInt(0, 1000);
        valueAnimator.setDuration(5000);
        valueAnimator.setInterpolator(new DecelerateInterpolator());//设置插值器
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {<!-- -->
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {<!-- -->
                tv1.setText(animation.getAnimatedValue() + "");
            }
        });
        valueAnimator.start();

```

>  
 插值器类型：（来自官方文档） 
 - AccelerateDecelerateInterpolator：开始和结束的时候慢，中间快- AccelerateInterpolator：开始的时候慢，然后加速- AnticipateInterpolator：开始先后退，然后向前- AnticipateOvershootInterpolator： 开始先后退，然向前到超标，最后回到最终值- BounceInterpolator ：最后会反弹- CycleInterpolator：动画会重复一定的周期数- DecelerateInterpolator：开始快，然后减速- LinearInterpolator：变化匀速- OvershootInterpolator：到达最终值后超标，再回到最终值- TimeInterpolator：用来自定义插值器 


# demo代码：

```
public class MainActivity extends AppCompatActivity {<!-- -->

    @Override
    protected void onCreate(Bundle savedInstanceState) {<!-- -->
        super.onCreate(savedInstanceState);
        LinearLayout linearLayout = new LinearLayout(this);
        linearLayout.setOrientation(LinearLayout.VERTICAL);
        setContentView(linearLayout);

        // test View
        final TextView tv1 = new TextView(this);
        tv1.setText(0 + "");
        tv1.setTextSize(30);
        linearLayout.addView(tv1);
        final Button btn0 = new Button(this);
        btn0.setText("btn0");
        linearLayout.addView(btn0);
        final Button btn1 = new Button(this);
        btn1.setText("btn1");
        linearLayout.addView(btn1);
        final TextView tv2 = new TextView(this);
        tv2.setText("tv2");
        tv2.setTextSize(30);
        linearLayout.addView(tv2);
        final Button btn2 = new Button(this);
        btn2.setText("btn2");
        linearLayout.addView(btn2);
        final Button btn3 = new Button(this);
        btn3.setText("btn3");
        linearLayout.addView(btn3);


        /*
         * ValueAnimator  +时间插值器演示
         */
        ValueAnimator valueAnimator = ValueAnimator.ofInt(0, 1000);
        valueAnimator.setDuration(5000);
        valueAnimator.setInterpolator(new DecelerateInterpolator());//设置插值器
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {<!-- -->
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {<!-- -->
                tv1.setText(animation.getAnimatedValue() + "");
            }
        });
        valueAnimator.start();

        /*
          ObjectAnimator + PropertyValuesHolder
         */
        PropertyValuesHolder xHolder = PropertyValuesHolder.ofFloat("scaleX", 1, 1.5f, 0.8f, 1);
        PropertyValuesHolder yHolder = PropertyValuesHolder.ofFloat("scaleY", 1, 1.5f, 0.8f, 1);
        ObjectAnimator objectAnimator = ObjectAnimator.ofPropertyValuesHolder(btn0, xHolder, yHolder);
        objectAnimator.setDuration(3000);
//        objectAnimator.setRepeatCount(-1);
        objectAnimator.start();

        /*
         * ObjectAnimator + AnimatorSet
         */
        ObjectAnimator anim1 = ObjectAnimator.ofFloat(btn1, "scaleX", 1, 1.5f, 0.8f, 1);
        ObjectAnimator anim2 = ObjectAnimator.ofFloat(btn1, "scaleY", 1, 1.5f, 0.8f, 1);
        final AnimatorSet animatorSet = new AnimatorSet();
        animatorSet.play(anim1);
        animatorSet.play(anim2).with(anim1);
        animatorSet.setDuration(3000);
        animatorSet.addListener(new Animator.AnimatorListener() {<!-- -->
            @Override
            public void onAnimationStart(Animator animation) {<!-- -->

            }

            @Override
            public void onAnimationEnd(Animator animation) {<!-- -->
                if (animatorSet != null) {<!-- -->
                    animatorSet.start();
                }
            }

            @Override
            public void onAnimationCancel(Animator animation) {<!-- -->

            }

            @Override
            public void onAnimationRepeat(Animator animation) {<!-- -->

            }
        });
        animatorSet.start();

        /*
         * ObjectAnimator + AnimatorSet
         */
        ObjectAnimator anim11 = ObjectAnimator.ofFloat(tv2, "translationX", 200);
        ObjectAnimator anim22 = ObjectAnimator.ofFloat(tv2, "scaleX", 1, 1.5f, 1f);
        ObjectAnimator anim33 = ObjectAnimator.ofFloat(tv2, "scaleY", 1, 1.5f, 1f);
        final AnimatorSet animatorSet2 = new AnimatorSet();
        animatorSet2.play(anim11);
        animatorSet2.play(anim22).after(anim11);
        animatorSet2.play(anim33).with(anim22);
        animatorSet2.setDuration(3000);
        animatorSet2.start();

    }

}

```
