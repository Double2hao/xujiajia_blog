#android动画如何更新UI（ValueAnimator源码解析）
# 概述

android动画经常会碰到卡顿，或者阻塞主进程之类的问题。 为了排查此类问题，不得不对动画原理了解一二，于是作此文。 此文围绕两个主线问题展开：
1. ui更新的频率是如何控制的？ 比如，1秒内会更新多少次？1. 每次更新UI的时候所带的update的value是如何控制的？ 比如，现在有个0到100的动画，在执行到30%的时候，value是多少？(可能非线性变化)
>  
 对动画还是不太了解的读者可以看下笔者之前的文章：  


# ValueAnimator源码

动画平时使用的API较多，但是最终的原理都是一样的，本文就选择从ValueAnimator的源码入手分析这两个问题。

### ui更新的频率

Demo代码如下：

```
    ValueAnimator valueAnimator = ValueAnimator.ofInt(0, 1000);
    valueAnimator.setDuration(2000);
    valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {<!-- -->
      @Override public void onAnimationUpdate(ValueAnimator animation) {<!-- -->
        Log.d("test", animation.getAnimatedValue() + "");
      }
    });
    valueAnimator.start();

```

直接在onAnimationUpdate方法中打断点，找到调用任务栈: <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2680.png" alt="在这里插入图片描述"> 最终是进入Choreographer的doFrame方法。

Choreographer的doFrame方法是在绘制每一帧的时候调用的，因此动画的更新频率与帧的更新频率一致。至于帧的更新频率是多少就看具体手机了。（每秒执行doFrame的次数其实就是fps）

### update的value如何控制？

首先看下getAnimatedValue源码：

```
    PropertyValuesHolder[] mValues;
    
    public Object getAnimatedValue() {<!-- -->
        if (mValues != null &amp;&amp; mValues.length &gt; 0) {<!-- -->
            return mValues[0].getAnimatedValue();
        }
        return null;
    }

```

由此我们可以知道，取的逻辑没有什么特殊的地方，其值在更新的时候早就设置好了，因此我们要找下设置的逻辑。 我们再使用ValueAnimation的时候，真正触发动画的逻辑其实是start()方法，因此我们从start()方法源码入手。经过如下调用，可以进入到设置value的地方:
- ValueAnimation.start()- ValueAnimation.start(boolean playBackwards)- ValueAnimation.setCurrentFraction(float fraction)- ValueAnimation.animateValue(float fraction)
```
    PropertyValuesHolder[] mValues;
    
    void animateValue(float fraction) {<!-- -->
        fraction = mInterpolator.getInterpolation(fraction);
        mCurrentFraction = fraction;
        int numValues = mValues.length;
        for (int i = 0; i &lt; numValues; ++i) {<!-- -->
            mValues[i].calculateValue(fraction);
        }
        if (mUpdateListeners != null) {<!-- -->
            int numListeners = mUpdateListeners.size();
            for (int i = 0; i &lt; numListeners; ++i) {<!-- -->
                mUpdateListeners.get(i).onAnimationUpdate(this);
            }
        }
    }

```

>  
 fraction可以认为是当前的值的百分比，比如我当前的值的范围是0~100，当前是120，那么fraction就是1.2f，如果当前是50，那么fraction就是0.5f。 


此处可以看到直接是将fraction传入到PropertyValuesHolder.calculateValue()方法来计算的value。

```
    Keyframes mKeyframes = null;
    void calculateValue(float fraction) {<!-- -->
        Object value = mKeyframes.getValue(fraction);
        mAnimatedValue = mConverter == null ? value : mConverter.convert(value);
    }

```

此处要了解Keyframes，可以从ValueAnimator.ofInt(int… values)从上至下看，也可以直接看PropertyValuesHolder中Keyframes的赋值来推导。 此处我们采用大家更熟悉的方式来，从ValueAnimator.ofInt(int… values)从上至下看：

```
    public static ValueAnimator ofInt(int... values) {<!-- -->
        ValueAnimator anim = new ValueAnimator();
        anim.setIntValues(values);
        return anim;
    }
    public void setIntValues(int... values) {<!-- -->
        if (values == null || values.length == 0) {<!-- -->
            return;
        }
        if (mValues == null || mValues.length == 0) {<!-- -->
            setValues(PropertyValuesHolder.ofInt("", values));
        } else {<!-- -->
            PropertyValuesHolder valuesHolder = mValues[0];
            valuesHolder.setIntValues(values);
        }
        // New property/values/target should cause re-initialization prior to starting
        mInitialized = false;
    }

```

```
    public void setIntValues(int... values) {<!-- -->
        mValueType = int.class;
        mKeyframes = KeyframeSet.ofInt(values);
    }

```

```
    public static KeyframeSet ofInt(int... values) {<!-- -->
        int numKeyframes = values.length;
        IntKeyframe keyframes[] = new IntKeyframe[Math.max(numKeyframes,2)];
        if (numKeyframes == 1) {<!-- -->
            keyframes[0] = (IntKeyframe) Keyframe.ofInt(0f);
            keyframes[1] = (IntKeyframe) Keyframe.ofInt(1f, values[0]);
        } else {<!-- -->
            keyframes[0] = (IntKeyframe) Keyframe.ofInt(0f, values[0]);
            for (int i = 1; i &lt; numKeyframes; ++i) {<!-- -->
                keyframes[i] =
                        (IntKeyframe) Keyframe.ofInt((float) i / (numKeyframes - 1), values[i]);
            }
        }
        return new IntKeyframeSet(keyframes);
    }

```

知道ValueAnimator.ofInt(int… values)其实就对应IntKeyframeSet之后，我们就可以以IntKeyframeSet为例来看下getValue的代码：

```
    @Override
    public Object getValue(float fraction) {<!-- -->
        return getIntValue(fraction);
    }

```

```
    public int getIntValue(float fraction) {<!-- -->
        if (fraction &lt;= 0f) {<!-- -->
            final IntKeyframe prevKeyframe = (IntKeyframe) mKeyframes.get(0);
            final IntKeyframe nextKeyframe = (IntKeyframe) mKeyframes.get(1);
            int prevValue = prevKeyframe.getIntValue();
            int nextValue = nextKeyframe.getIntValue();
            float prevFraction = prevKeyframe.getFraction();
            float nextFraction = nextKeyframe.getFraction();
            final TimeInterpolator interpolator = nextKeyframe.getInterpolator();
            if (interpolator != null) {<!-- -->
                fraction = interpolator.getInterpolation(fraction);
            }
            float intervalFraction = (fraction - prevFraction) / (nextFraction - prevFraction);
            return mEvaluator == null ?
                    prevValue + (int)(intervalFraction * (nextValue - prevValue)) :
                    ((Number)mEvaluator.evaluate(intervalFraction, prevValue, nextValue)).
                            intValue();
        } else if (fraction &gt;= 1f) {<!-- -->
            final IntKeyframe prevKeyframe = (IntKeyframe) mKeyframes.get(mNumKeyframes - 2);
            final IntKeyframe nextKeyframe = (IntKeyframe) mKeyframes.get(mNumKeyframes - 1);
            int prevValue = prevKeyframe.getIntValue();
            int nextValue = nextKeyframe.getIntValue();
            float prevFraction = prevKeyframe.getFraction();
            float nextFraction = nextKeyframe.getFraction();
            final TimeInterpolator interpolator = nextKeyframe.getInterpolator();
            if (interpolator != null) {<!-- -->
                fraction = interpolator.getInterpolation(fraction);
            }
            float intervalFraction = (fraction - prevFraction) / (nextFraction - prevFraction);
            return mEvaluator == null ?
                    prevValue + (int)(intervalFraction * (nextValue - prevValue)) :
                    ((Number)mEvaluator.evaluate(intervalFraction, prevValue, nextValue)).intValue();
        }
        IntKeyframe prevKeyframe = (IntKeyframe) mKeyframes.get(0);
        for (int i = 1; i &lt; mNumKeyframes; ++i) {<!-- -->
            IntKeyframe nextKeyframe = (IntKeyframe) mKeyframes.get(i);
            if (fraction &lt; nextKeyframe.getFraction()) {<!-- -->
                final TimeInterpolator interpolator = nextKeyframe.getInterpolator();
                float intervalFraction = (fraction - prevKeyframe.getFraction()) /
                    (nextKeyframe.getFraction() - prevKeyframe.getFraction());
                int prevValue = prevKeyframe.getIntValue();
                int nextValue = nextKeyframe.getIntValue();
                // Apply interpolator on the proportional duration.
                if (interpolator != null) {<!-- -->
                    intervalFraction = interpolator.getInterpolation(intervalFraction);
                }
                return mEvaluator == null ?
                        prevValue + (int)(intervalFraction * (nextValue - prevValue)) :
                        ((Number)mEvaluator.evaluate(intervalFraction, prevValue, nextValue)).
                                intValue();
            }
            prevKeyframe = nextKeyframe;
        }
        // shouldn't get here
        return ((Number)mKeyframes.get(mNumKeyframes - 1).getValue()).intValue();
    }

```

一般情况下，如果是线性插值器的话，fraction其实就只代表进度，那么fraction是肯定&gt;0且&lt;1的，我们要看的代码其实只有这一段：

```
        IntKeyframe prevKeyframe = (IntKeyframe) mKeyframes.get(0);
        for (int i = 1; i &lt; mNumKeyframes; ++i) {<!-- -->
            IntKeyframe nextKeyframe = (IntKeyframe) mKeyframes.get(i);
            if (fraction &lt; nextKeyframe.getFraction()) {<!-- -->
                final TimeInterpolator interpolator = nextKeyframe.getInterpolator();
                float intervalFraction = (fraction - prevKeyframe.getFraction()) /
                    (nextKeyframe.getFraction() - prevKeyframe.getFraction());
                int prevValue = prevKeyframe.getIntValue();
                int nextValue = nextKeyframe.getIntValue();
                // Apply interpolator on the proportional duration.
                if (interpolator != null) {<!-- -->
                    intervalFraction = interpolator.getInterpolation(intervalFraction);
                }
                return mEvaluator == null ?
                        prevValue + (int)(intervalFraction * (nextValue - prevValue)) :
                        ((Number)mEvaluator.evaluate(intervalFraction, prevValue, nextValue)).
                                intValue();
            }
            prevKeyframe = nextKeyframe;

```

此时用一个例子来看就非常简单：

```
ValueAnimator.ofInt(0,-20,120,40);

```

此时，这几个值对应的fraction分别是：(这个可以通过上面KeyframeSet.ofInt()的源码得到)
- 0：0- -20：1/3- 120: 2/3- 40: 1 因此，如果此时fraction为0.5的话，1/3&lt;0.5&lt;2/3 那么此时nextValue就为120,prevValue就为-20。
至于具体的返回值，分别需要由插值器或者mEvaluator来决定。

>  
 自定义插值器或者其他一些特殊的情况此处就不增加篇幅了。 有了代码，逻辑也很清晰，读者可以自己考虑下。 
