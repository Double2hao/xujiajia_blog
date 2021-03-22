#快速理解android View的测量onMeasure()与MeasureSpec
笔者之前有一篇文章已经使用onMeasure()解决了listview与scollview的显示冲突问题，博客地址如下：



 

在此就针对View的测量以及onMeasure()涉及的几个问题做一个详细解释：

一、MeasureSpec的概念：

MeasureSpec通过将SpecMode和SpecSize打包成一个int值来避免过多的对象内存分配，为了方便操作，其提供了打包和解包的方法。SpecMode和SpecSize也是一个int值，一组SpecMode和SpecSize可以打包为一个MeasureSpec，而一个MeasureSpec可以通过解包的形式来得出其原始的SpecMode和SpecSize。

读者只要记住以下一句话即可：

**MeasureSpec的值由specSize和specMode共同组成的，其中specSize记录的是大小，specMode记录的是规格。**

 

二、SpecMode的三种模式：

 

1. **EXACTLY**

当我们将控件的“layout_width”属性或者“layout_height”属性指定为具体数值时，比如“android:layout_width="200dp"”，或者指定为“match_parent”时，系统会使用这个模式。

2. **AT_MOST**

当控件的“layout_width”属性或者“layout_height”属性设置为“wrap_content”时，控件大小一般会随着内容的大小而变化，但是无论多大，也不能超过父控件的尺寸。

3. **UNSPECIFIED**

表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，一般在绘制自定义View的时候才会用到。

 

三、View的测量到底和什么有关呢？

要探其原理，首先要和大家说明一点，一个View只需要MeasureSpec确定，那么在onMeasure中就可以测量它的宽高，所以我们可以将问题直接转化成“一个View的MeasureSpec是如何确定的呢？”

 

普通的View的measure过程由VIewGroup传递而来，此处我们根据源码来做一个解释，先看一下ViewGroup中的measureChildWithMargins()：

 

```
protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

从中，我们可以看到一个view的宽高，都是通过getChildMeasureSpec()这个方法获得的，那么这里面又是怎么实现的呢？我们不妨Control+左键点进去看一下，代码如下：

 

 

```
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension &gt;= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension &gt;= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension &gt;= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

 

代码比较长，读者可以比较焦急——不用害怕，我们不需要完全理解它的原理，我们只需要知道View的测量是如何实现的就行了。

看到源码方法中的三个参数，并且比较measureChildWithMargins()方法中传递给getChildMeasureSpec()三个值，我们很快就可以理解，**一个View的测量过程是由父布局的MeasureSpec和该View的LayoutParams决定的。**

 

读者可以看measureChildWithMargins()中如下代码：

 

```
final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
```

 

 

要测量子部局的宽度的MeasureSpec，需要传入3个参数：

第一个参数：父布局的宽度的MeasureSpec

第二个参数：子部局的padding值，子部局的LayoutParams的Margin值

第三个参数：子部局的LayoutParams的宽度

 

 

 

如果对View的测量过程由更深入的求知欲的，推荐读者可以自己看一下源码。