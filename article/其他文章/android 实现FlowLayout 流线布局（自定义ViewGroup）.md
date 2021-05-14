#android 实现FlowLayout 流线布局（自定义ViewGroup）
# 项目目的

1、深化自定义View的概念 2、将MeasureSpec、View的绘制流程、Layoutparams等分散的知识点整合成一个demo。

# 项目灵感

笔者经验也是有限，此文章主要借鉴张鸿洋前辈的博客。 原文地址：

# 项目预览（源码附文章结尾）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910451430.png " alt="这里写图片描述">

# 主要涉及知识点以及相关链接

### 1、MeasureSpec



### 2、Layoutparams以及MarginLayoutParams

 

### 3、View的绘制流程



### 4、笔者自定View相关文章

 

# 实现要点

1、重写onMeasure()方法，让FlowLayout能够根据子部局的大小来确定自己的大小。 2、重写onLayout()方法，让FlowLayout中的子部局能够即正确的排版，即对其间的子View进行布局。 3、使用MarginLayoutParams，不仅能够获取到子部局的宽高，还能获取到子布局的margin参数。

# 主要代码

```
public class FlowLayout extends ViewGroup {

    private static final String TAG = "FlowLayout";

    public FlowLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected ViewGroup.LayoutParams generateLayoutParams(
            ViewGroup.LayoutParams p) {
        return new MarginLayoutParams(p);
    }

    @Override
    public ViewGroup.LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new MarginLayoutParams(getContext(), attrs);
    }

    @Override
    protected ViewGroup.LayoutParams generateDefaultLayoutParams() {
        return new MarginLayoutParams(LayoutParams.MATCH_PARENT,
                LayoutParams.MATCH_PARENT);
    }

    // 负责设置子控件的测量模式和大小 根据所有子控件设置自己的宽和高
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        // 获得它的父容器为它设置的测量模式和大小
        int sizeWidth = MeasureSpec.getSize(widthMeasureSpec);
        int sizeHeight = MeasureSpec.getSize(heightMeasureSpec);
        int modeWidth = MeasureSpec.getMode(widthMeasureSpec);
        int modeHeight = MeasureSpec.getMode(heightMeasureSpec);

        Log.e(TAG, sizeWidth + "," + sizeHeight);

        // 如果是warp_content情况下，记录宽和高
        int width = 0;
        int height = 0;

        int lineWidth = 0;//每行的宽度
        int lineHeight = 0;//每行的高度

        int cCount = getChildCount();
        // 遍历每个子元素
        for (int i = 0; i &lt; cCount; i++) {
            View child = getChildAt(i);
            // 测量每一个child的宽和高
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
            // 得到child的lp
            MarginLayoutParams lp = (MarginLayoutParams) child
                    .getLayoutParams();
            int childWidth = child.getMeasuredWidth() + lp.leftMargin
                    + lp.rightMargin;// 当前子空间实际占据的宽度
            int childHeight = child.getMeasuredHeight() + lp.topMargin
                    + lp.bottomMargin;// 当前子空间实际占据的高度

            //如果加入当前child，超出最大宽度，则增加高度
            if (lineWidth + childWidth &gt; sizeWidth) {
                width = Math.max(lineWidth, childWidth);// 取最大的
                lineWidth = childWidth; // 重新开启新行，开始记录
                height += lineHeight;// 叠加当前高度，
                lineHeight = childHeight;// 开启记录下一行的高度
            } else
            // 否则累加值lineWidth,lineHeight取最大高度
            {
                lineWidth += childWidth;
                lineHeight = Math.max(lineHeight, childHeight);
            }
            // 如果是最后一个，则将当前记录的最大宽度和当前lineWidth做比较
            if (i == cCount - 1) {
                width = Math.max(width, lineWidth);
                height += lineHeight;
            }

        }
        //设置布局宽高有三种情况
        //如果是wrap_content，则根据所有子部局的大小来显示
        //如果是确切值，就根据确切值的大小显示
        //如果是match_parent，则显示父布局能显示的最大值
        setMeasuredDimension((modeWidth == MeasureSpec.EXACTLY) ? sizeWidth
                : width, (modeHeight == MeasureSpec.EXACTLY) ? sizeHeight
                : height);

    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int width = getWidth();

        int lineWidth = 0;
        int lineHeight = 0;
        // 获取孩子的个数
        int cCount = getChildCount();

        int left = 0;
        int top = 0;
        // 遍历所有的孩子,
        for (int i = 0; i &lt; cCount; i++) {
            View child = getChildAt(i);
            MarginLayoutParams lp = (MarginLayoutParams) child
                    .getLayoutParams();

            int childWidth = child.getMeasuredWidth();
            int childHeight = child.getMeasuredHeight();
            // 如果已经需要换行
            if (childWidth + lp.leftMargin + lp.rightMargin + lineWidth &gt; width) {
                lineWidth = 0;// 重置行宽
                left = 0;
                top += lineHeight;
            }

            //计算childView的left,top,right,bottom,并且设置每个View的位置
            int lc = left + lp.leftMargin;
            int tc = top + lp.topMargin;
            int rc = lc + child.getMeasuredWidth();
            int bc = tc + child.getMeasuredHeight();
            child.layout(lc, tc, rc, bc);

            //最后不要忘记累加
            left += child.getMeasuredWidth() + lp.rightMargin
                    + lp.leftMargin;
            lineWidth += childWidth + lp.leftMargin + lp.rightMargin;
            lineHeight = Math.max(lineHeight, childHeight + lp.topMargin
                    + lp.bottomMargin);
        }
    }
}

```

源码下载：