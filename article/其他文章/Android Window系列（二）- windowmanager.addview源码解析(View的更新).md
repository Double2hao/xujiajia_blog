#Android Window系列（二）- windowmanager.addview源码解析(View的更新)
# 概述

前文讲解了window与decorview相关的知识点，有兴趣的读者可以看下： 

本文将继续探讨下window与view的关系，主要针对“如何在window中添加view”来进行探索。

# 如何在window中添加View

这样的场景有非常多，有如下例子：
- activity在启动的时候向window中添加view- dialog在启动的时候向window中添加view- toast在启动的时候向window中添加view- 悬浮窗
### activity在启动的时候向window中添加view

相关逻辑的任务栈调用如下：
- ActivityThread.handleResumeActivity()- WindowManager.addView()
>  
 此处也可以解释为什么在onCreate的时候去获取view的宽高为什么会不准确。 原因是，在执行onResume之前才会将View添加到Window上。 


### 悬浮窗

笔者之前写过相关的文章，有兴趣的读者可以看下： 

笔者demo中悬浮窗展示的关键代码如下：

```
  @UiThread
  public void showDebugViewOnUiThread(Context context) {<!-- -->
    if (studyFloatUtilView != null || delegate == null || !isOpenFloatUtil) {<!-- -->
      return;
    }
    if (Build.VERSION.SDK_INT &gt;= Build.VERSION_CODES.M &amp;&amp; !Settings.canDrawOverlays(context)) {<!-- -->
      Toast.makeText(context, "SoGameDebug功能需要打开悬浮窗权限才能使用", Toast.LENGTH_LONG).show();
      Intent intent = new Intent();
      intent.setAction(Settings.ACTION_MANAGE_OVERLAY_PERMISSION);
      intent.setData(Uri.parse("package:" + context.getPackageName()));
      context.startActivity(intent);
    }
    //宽高设置为屏幕宽度。（游戏存在横屏与竖屏）
    WindowManager.LayoutParams layoutParams =
        new WindowManager.LayoutParams();
    layoutParams.height = WindowManager.LayoutParams.WRAP_CONTENT;
    layoutParams.width = WindowManager.LayoutParams.WRAP_CONTENT;
    if (Build.VERSION.SDK_INT &gt;= Build.VERSION_CODES.O) {<!-- -->
      layoutParams.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY;
    } else {<!-- -->
      layoutParams.type = WindowManager.LayoutParams.TYPE_PHONE;
    }
    layoutParams.format = PixelFormat.RGBA_8888;
    layoutParams.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL
        | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE;
    layoutParams.gravity = Gravity.END | Gravity.CENTER_VERTICAL;

    studyFloatUtilView = new StudyFloatUtilView(context);
    WindowManager windowManager = (WindowManager) context.getSystemService(WINDOW_SERVICE);
    if (windowManager == null) {<!-- -->
      return;
    }
    windowManager.addView(studyFloatUtilView, layoutParams);
  }

```

# WindowManagerImpl官方描述

由于WindowManager本身是interface类型，因此我们直接看到它的实现类WindowManagerImpl。

```
/**
 * Provides low-level communication with the system window manager for
 * operations that are bound to a particular context, display or parent window.
 * Instances of this object are sensitive to the compatibility info associated
 * with the running application.
 *
 * This object implements the {@link ViewManager} interface,
 * allowing you to add any View subclass as a top-level window on the screen.
 * Additional window manager specific layout parameters are defined for
 * control over how windows are displayed.  It also implements the {@link WindowManager}
 * interface, allowing you to control the displays attached to the device.
 *
 * &lt;p&gt;Applications will not normally use WindowManager directly, instead relying
 * on the higher-level facilities in {@link android.app.Activity} and
 * {@link android.app.Dialog}.
 *
 * &lt;p&gt;Even for low-level window manager access, it is almost never correct to use
 * this class.  For example, {@link android.app.Activity#getWindowManager}
 * provides a window manager for adding windows that are associated with that
 * activity -- the window manager will not normally allow you to add arbitrary
 * windows that are not associated with an activity.
 *
 * @see WindowManager
 * @see WindowManagerGlobal
 * @hide
 */

```

笔者整理后大致内容如下：
1. 提供与底层系统window manager 的通信方式。1. WindowManagerImpl实现了ViewManager接口，用来在顶部的window中添加view。1. windowManager的其他参数用来控制window的展示。WindowManagerImpl实现了WindowManager接口，用来控制设备界面的展示。1. 应用一般情况下不会直接使用WindowManager，一般还是用Activity和Dialog。1. 如果直接使用windowManager的话，很难保证使用不犯错。比如通过Activity.getWindowManager()拿到的WindowManager，就不允许添加和这个activity无关的window。
# windowmanager.addview()

window与view关系的关键就是windowmanager.addview()这个方法。 先列出调用流程，让读者能有个大概的认知：
- windowmanager.addview()- WindowManagerImpl.addview()- WindowManagerGlobal.addview()- ViewRootImpl.setView()
# WindowManagerImpl.addview()

WindowManagerImpl.addView()总最终是调用到了WindowManagerGlobal的addView()方法。

```
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();

    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {<!-- -->
        applyDefaultToken(params);
        mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
    }

```

# WindowManagerGlobal官方描述

```
/**
 * Provides low-level communication with the system window manager for
 * operations that are not associated with any particular context.
 *
 * This class is only used internally to implement global functions where
 * the caller already knows the display and relevant compatibility information
 * for the operation.  For most purposes, you should use {@link WindowManager} instead
 * since it is bound to a context.
 *
 * @see WindowManagerImpl
 * @hide
 */

```

笔者整理后大致内容如下：
1. 为context提供了与底层系统window manager的相关接口。1. 这个类一般是用来给内部持有display 和相关兼容信息的对象调用的。1. 对于大多数情况，开发者不应该直接使用WindowManagerGlobal，而是使用绑定了context的windowManager.
# WindowManagerGlobal.addView()

此方法代码较多，我们就看与探索的逻辑相关的代码：
1. 创建ViewRootImpl，缓存到arrayList中。1. 将view 与对应WindowManager.LayoutParams 都缓存到arrayList中。1. 调用ViewRootImpl.setView方法，触发ui更新。
```

    private final ArrayList&lt;View&gt; mViews = new ArrayList&lt;View&gt;();
    private final ArrayList&lt;ViewRootImpl&gt; mRoots = new ArrayList&lt;ViewRootImpl&gt;();
    private final ArrayList&lt;WindowManager.LayoutParams&gt; mParams =
            new ArrayList&lt;WindowManager.LayoutParams&gt;();

    public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {<!-- -->
————————————————省略

        ViewRootImpl root;
        
————————————————省略
            root = new ViewRootImpl(view.getContext(), display);

            view.setLayoutParams(wparams);

            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);

            // do this last because it fires off messages to start doing things
            try {<!-- -->
                root.setView(view, wparams, panelParentView);
            } catch (RuntimeException e) {<!-- -->
                // BadTokenException or InvalidDisplayException, clean up.
                if (index &gt;= 0) {<!-- -->
                    removeViewLocked(index, true);
                }
                throw e;
            }
        }
    }

```

# ViewRootImpl.setView()

ViewRootImpl.setView的源码较多，我们就只留下与View相关的这一部分。 此处主要是两个逻辑：
1. ViewRootImpl.requestLayout() 更新UI的逻辑1. IWindowSession.addToDisplay() 添加window到系统中
```
    final IWindowSession mWindowSession;

    public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {<!-- -->
————————————————省略
                requestLayout();
————————————————省略
                    res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
                            getHostVisibility(), mDisplay.getDisplayId(),
                            mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
                            mAttachInfo.mOutsets, mInputChannel);
————————————————省略
    }

```

# ViewRootImpl.requestLayout()

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

### 继续讲下activity与window

前文讲到activity中调用到windowManager的任务栈是：
- ActivityThread.handleResumeActivity()- WindowManager.addView()
由此逻辑，我们也可以确定activity中View第一次更新UI是在handleResumeActivity()中。

也因此，当我们在onCreate中去获取View的宽高等属性经常会等于0,因为在onCreate中，View更新UI的逻辑还没有执行。

>  
 如果想要再OnCreate中添加 “在UI更新结束之后”的逻辑可以使用ViewTreeObserver。 对此有兴趣的读者可以关注笔者的另一篇文章：  


# IWindowSession.addToDisplay()

addToDisplay()的调用流程较长，此处直接列出调用流程：
- IWindowSession.addToDisplay() (跨进程调用)- Session.addToDisplay()- WindowManagerService.addWindow()- PhoneWindowManager.prepareAddWindowLw()
Session继承了IWindowSession.Stub，由可知IWindowSession.addToDisplay()是通过IPC，最终在系统进程使用WindowManagerService.addWindow()来实现了更新window的逻辑。

>  
 讲下笔者的个人理解，如有问题欢迎评论指正。 View的所有更新UI的操作最终都必须经过操作系统在系统进程的处理，才能够通过硬件展示到用户面前。 也因此ViewRootImpl是非常重要的一部分，View可以通过ViewRootImpl将更新UI的操作告知操作系统。而IWindowSession.addToDisplay()则是其中关键的一个调用。 


此处就不继续深入了，对相关源码有兴趣的读者可以自行看下。

# ViewRootImpl继续探索

细心的读者会发现对于ViewRootImpl还是有很多没有讲明白的点：
- ViewRootImpl与View究竟有哪些关系？- ViewRootImpl有哪些作用？
对此有兴趣的读者欢迎关注读者后续文章，或者也可以自行阅读下源码。