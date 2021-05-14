#ViewRootImpl源码解析 (一) - View的更新
# 概述

前文讲解了windowManager.addView() 源码相关的知识，有兴趣的读者可以看下： 

前文讲到ViewRootImpl，但是属于浅尝辄止，没有继续深入了，因此笔者再开几篇文章，继续深入讲下ViewRootImpl的源码。

>  
 此处先列出笔者的个人对ViewRootImpl的理解，如有问题欢迎评论指正。 如下： View的所有更新UI的操作最终都必须经过操作系统在系统进程的处理，才能够通过硬件展示到用户面前。 ViewRootImpl担任了window与view的中间人的角色，View可以通过ViewRootImpl将更新UI的操作告知操作系统，而操作系统也可以将触摸等硬件层面操作通过ViewRootImpl反馈给View。 


# ViewRootImpl官方解释

```
/**
 * The top of a view hierarchy, implementing the needed protocol between View
 * and the WindowManager.  This is for the most part an internal implementation
 * detail of {@link WindowManagerGlobal}.
 *
 * {@hide}
 */

```

笔者整理后要点如下：
1. view层级的最顶层，实现了View与WindowManager之间所有需要用到的接口。1. ViewRootImpl这个类是WindowManagerGlobal中非常重要的一部分。
# ViewRootImpl.requestLayout()

此处列举两个requestLayout()常用的地方：
1.  给window添加view最终会调用到ViewRootImpl.requestLayout() 对这部分有兴趣的可以看下前文：  1.  View.setLayoutParams()最终会调用到ViewRootImpl.requestLayout() 调用流程如下：View.setLayoutParams() -&gt; View.requestLayout() -&gt; ViewRootImpl.requestLayout() 
### 官方注释

requestLayout是ViewParent接口中的方法，其中的注释解释如下：

```
    /**
     * Called when something has changed which has invalidated the layout of a
     * child of this view parent. This will schedule a layout pass of the view
     * tree.
     */

```

笔者整理后意思如下：
1. 当这个view的parent中UI有所变动时，会调用这个方法。1. 这个方法会重新遍历整个view的布局。
### 调用栈

requestLayout()后续调用的流程较长，此处直接列出调用栈，有兴趣的读者可以自行阅读下源码。
- ViewRootImpl.requestLayout()- ViewRootImpl.scheduleTraversals()- ViewRootImpl.doTraversal()- ViewRootImpl.performTraversals()
performTraversals()源码如下。 其中的三个方法最终会调用到view的方法，对应关系如下：
- performMeasure()最终会调用到View.measure()- performLayout()最终会调用到View.layout()- performDraw()最终会调用到View.draw()
```
private void performTraversals() {<!-- -->  
————————————————省略
        performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
————————————————省略
        performLayout(lp, desiredWindowWidth, desiredWindowHeight);
————————————————省略
        performDraw();
————————————————省略
        }
        ......  
    }  
       

```

至此，我们了解到在windowManager调用了addview之后，最终会走到View更新的逻辑。

# View.invalidate()

View.invalidate()是在自定义View、RecyclerView等中常用的一个方法，使用它可以触发View的重绘。 它最终其实也是调用到ViewRootImpl，此处就深入讨论下。

### 官方注释

```
    /**
     * Invalidate the whole view. If the view is visible,
     * {@link #onDraw(android.graphics.Canvas)} will be called at some point in
     * the future.
     * &lt;p&gt;
     * This must be called from a UI thread. To call from a non-UI thread, call
     * {@link #postInvalidate()}.
     */

```

笔者理解的翻译如下：

>  
 当View可见时，invalidate()会让整个View失效并重绘，最终会触发到这个View的onDraw()方法。 (此处如果是ViewGroup的话，会触发它所有子View的onDraw()) 


### 调用栈
- View.invalidate()- View.invalidateInternal()- ViewRootImpl.invalidateChild()- ViewRootImpl.invalidateChildInParent()- ViewRootImpl.invalidate()- ViewRootImpl.scheduleTraversals()- ViewRootImpl.doTraversal()- ViewRootImpl.performTraversals()- View.onDraw()
最终会在ViewRootImpl.performTraversals()中调用到View的onDraw()方法。