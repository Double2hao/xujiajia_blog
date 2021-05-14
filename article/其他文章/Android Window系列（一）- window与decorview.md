#Android Window系列（一）- window与decorview
# 概述

window是android中非常常见的一个概念。Activity、Dialog、Toast这些常用的知识点都是和window密不可分的。 因此，笔者整理了下window相关的知识，期望能对需要的读者有所帮助。

# window官方描述

Window源码中对window的描述如下：

```
/**
 * Abstract base class for a top-level window look and behavior policy.  An
 * instance of this class should be used as the top-level view added to the
 * window manager. It provides standard UI policies such as a background, title
 * area, default key processing, etc.
 *
 * &lt;p&gt;The only existing implementation of this abstract class is
 * android.view.PhoneWindow, which you should instantiate when needing a
 * Window.
 */

```

笔者翻译后的大致意思如下：
- window是一个抽象类，主要用来处理窗口的展示与行为策略（比如触摸，点击等）。- window类的实例应该是一个被添加到windowManager的顶级视图。- window提供了标准的UI策略，如背景，标题区域，默认密钥处理等。- widnow唯一的实现类是android.view.PhoneWindow，如果要使用window就必须通过android.view.PhoneWindow。
# window与decorview

window与decorview的逻辑关系如下：
- 一个Activity对应一个PhoneWindow，PhoneWidnow会处理这个activity中的ui展示和 用户的行为（如触摸，点击等）。- PhoneWidnow不是一个View对象，通过将PhoneWindow添加到windowManager中，PhoneWindow能够将要处理的行为事件传递给DecorView。- DecorView继承自FrameLayout，是除了Window之外最顶级的视图。- ContentView就是我们通常使用activity.setContentView()中设置的View。它所对应的id是R.id.content。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039773740.png" alt="在这里插入图片描述">
# 源码解析

接下来就通过源码解析看下具体的window相关的层级之间的关系。

### Activity.setContentView（）

逻辑如下：
1. 调用window.setContentView() 来初始化Layout1. 调用initWindowDecorActionBar来初始化 actionBar
由于是关注window相关层级之间的关系，因此接下来直接看window.setContentView的源码。

```
    public void setContentView(@LayoutRes int layoutResID) {<!-- -->
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }

```

### PhoneWindow.setContentView()

根据上文解析的Window的注释，已知window的实例是PhoneWindow，因此直接跳转到PhoneWindow的源码。

主要逻辑如下：
1. 首先判断contentView是否已经创建，如果没有创建就需要调用installDecor()来初始化DecorView。1. 然后会调用inflate()方法，将layout添加到contentView中。（FEATURE_CONTENT_TRANSITIONS是转场动画相关逻辑，暂且不看。）1. 最后调用window的callBack的onContentChanged()方法，通知window内容的更改。
```
    // This is the view in which the window contents are placed. It is either
    // mDecor itself, or a child of mDecor where the contents go.
    ViewGroup mContentParent;

    public void setContentView(int layoutResID) {<!-- -->
        if (mContentParent == null) {<!-- -->
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {<!-- -->
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {<!-- -->
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {<!-- -->
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null &amp;&amp; !isDestroyed()) {<!-- -->
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }

```

### PhoneWindow.installDecor()

installDecor()是初始化DecorView的相关逻辑，主要逻辑如下：
1. 判断DecorView是否为空，如果为空就调用generateDecor()来创建。如果已经初始化过了，那么就再调用setWindow绑定一下此时的window。1. 判断contentView是否为空，如果为空，就调用generateLayout()来创建。1. decorContentParent就是toolbar的View，也是通过判断是否为空来执行初始化逻辑。1. 而后的FEATURE_ACTIVITY_TRANSITIONS就是转场动画相关的逻辑，暂且不看。
```
    private void installDecor() {<!-- -->
        mForceDecorInstall = false;
        if (mDecor == null) {<!-- -->
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted &amp;&amp; mInvalidatePanelMenuFeatures != 0) {<!-- -->
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {<!-- -->
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {<!-- -->
            mContentParent = generateLayout(mDecor);

            // Set up decor part of UI to ignore fitsSystemWindows if appropriate.
            mDecor.makeOptionalFitsSystemWindows();

            final DecorContentParent decorContentParent = (DecorContentParent) mDecor.findViewById(
                    R.id.decor_content_parent);

            if (decorContentParent != null) {<!-- -->
                mDecorContentParent = decorContentParent;
                mDecorContentParent.setWindowCallback(getCallback());
                if (mDecorContentParent.getTitle() == null) {<!-- -->
                    mDecorContentParent.setWindowTitle(mTitle);
                }

                final int localFeatures = getLocalFeatures();
                for (int i = 0; i &lt; FEATURE_MAX; i++) {<!-- -->
                    if ((localFeatures &amp; (1 &lt;&lt; i)) != 0) {<!-- -->
                        mDecorContentParent.initFeature(i);
                    }
                }

                mDecorContentParent.setUiOptions(mUiOptions);

                if ((mResourcesSetFlags &amp; FLAG_RESOURCE_SET_ICON) != 0 ||
                        (mIconRes != 0 &amp;&amp; !mDecorContentParent.hasIcon())) {<!-- -->
                    mDecorContentParent.setIcon(mIconRes);
                } else if ((mResourcesSetFlags &amp; FLAG_RESOURCE_SET_ICON) == 0 &amp;&amp;
                        mIconRes == 0 &amp;&amp; !mDecorContentParent.hasIcon()) {<!-- -->
                    mDecorContentParent.setIcon(
                            getContext().getPackageManager().getDefaultActivityIcon());
                    mResourcesSetFlags |= FLAG_RESOURCE_SET_ICON_FALLBACK;
                }
                if ((mResourcesSetFlags &amp; FLAG_RESOURCE_SET_LOGO) != 0 ||
                        (mLogoRes != 0 &amp;&amp; !mDecorContentParent.hasLogo())) {<!-- -->
                    mDecorContentParent.setLogo(mLogoRes);
                }

                // Invalidate if the panel menu hasn't been created before this.
                // Panel menu invalidation is deferred avoiding application onCreateOptionsMenu
                // being called in the middle of onCreate or similar.
                // A pending invalidation will typically be resolved before the posted message
                // would run normally in order to satisfy instance state restoration.
                PanelFeatureState st = getPanelState(FEATURE_OPTIONS_PANEL, false);
                if (!isDestroyed() &amp;&amp; (st == null || st.menu == null) &amp;&amp; !mIsStartingWindow) {<!-- -->
                    invalidatePanelMenu(FEATURE_ACTION_BAR);
                }
            } else {<!-- -->
                mTitleView = findViewById(R.id.title);
                if (mTitleView != null) {<!-- -->
                    if ((getLocalFeatures() &amp; (1 &lt;&lt; FEATURE_NO_TITLE)) != 0) {<!-- -->
                        final View titleContainer = findViewById(R.id.title_container);
                        if (titleContainer != null) {<!-- -->
                            titleContainer.setVisibility(View.GONE);
                        } else {<!-- -->
                            mTitleView.setVisibility(View.GONE);
                        }
                        mContentParent.setForeground(null);
                    } else {<!-- -->
                        mTitleView.setText(mTitle);
                    }
                }
            }

            if (mDecor.getBackground() == null &amp;&amp; mBackgroundFallbackResource != 0) {<!-- -->
                mDecor.setBackgroundFallback(mBackgroundFallbackResource);
            }

            // Only inflate or create a new TransitionManager if the caller hasn't
            // already set a custom one.
            if (hasFeature(FEATURE_ACTIVITY_TRANSITIONS)) {<!-- -->
                if (mTransitionManager == null) {<!-- -->
                    final int transitionRes = getWindowStyle().getResourceId(
                            R.styleable.Window_windowContentTransitionManager,
                            0);
                    if (transitionRes != 0) {<!-- -->
                        final TransitionInflater inflater = TransitionInflater.from(getContext());
                        mTransitionManager = inflater.inflateTransitionManager(transitionRes,
                                mContentParent);
                    } else {<!-- -->
                        mTransitionManager = new TransitionManager();
                    }
                }

                mEnterTransition = getTransition(mEnterTransition, null,
                        R.styleable.Window_windowEnterTransition);
                mReturnTransition = getTransition(mReturnTransition, USE_DEFAULT_TRANSITION,
                        R.styleable.Window_windowReturnTransition);
                mExitTransition = getTransition(mExitTransition, null,
                        R.styleable.Window_windowExitTransition);
                mReenterTransition = getTransition(mReenterTransition, USE_DEFAULT_TRANSITION,
                        R.styleable.Window_windowReenterTransition);
                mSharedElementEnterTransition = getTransition(mSharedElementEnterTransition, null,
                        R.styleable.Window_windowSharedElementEnterTransition);
                mSharedElementReturnTransition = getTransition(mSharedElementReturnTransition,
                        USE_DEFAULT_TRANSITION,
                        R.styleable.Window_windowSharedElementReturnTransition);
                mSharedElementExitTransition = getTransition(mSharedElementExitTransition, null,
                        R.styleable.Window_windowSharedElementExitTransition);
                mSharedElementReenterTransition = getTransition(mSharedElementReenterTransition,
                        USE_DEFAULT_TRANSITION,
                        R.styleable.Window_windowSharedElementReenterTransition);
                if (mAllowEnterTransitionOverlap == null) {<!-- -->
                    mAllowEnterTransitionOverlap = getWindowStyle().getBoolean(
                            R.styleable.Window_windowAllowEnterTransitionOverlap, true);
                }
                if (mAllowReturnTransitionOverlap == null) {<!-- -->
                    mAllowReturnTransitionOverlap = getWindowStyle().getBoolean(
                            R.styleable.Window_windowAllowReturnTransitionOverlap, true);
                }
                if (mBackgroundFadeDurationMillis &lt; 0) {<!-- -->
                    mBackgroundFadeDurationMillis = getWindowStyle().getInteger(
                            R.styleable.Window_windowTransitionBackgroundFadeDuration,
                            DEFAULT_BACKGROUND_FADE_DURATION_MS);
                }
                if (mSharedElementsUseOverlay == null) {<!-- -->
                    mSharedElementsUseOverlay = getWindowStyle().getBoolean(
                            R.styleable.Window_windowSharedElementsUseOverlay, true);
                }
            }
        }
    }

```

### PhoneWindow.generateDecor()

generateDecor()是创建DecorView的相关逻辑，主要逻辑如下：
1. 先判断此时的context是否是DecorContext，如果不是，就通过application和此时的content来new 一个DecorContext。1. 直接new一个DecorView返回，需要传入参数DecorContext 和此时的window。
```
    protected DecorView generateDecor(int featureId) {<!-- -->
        // System process doesn't have application context and in that case we need to directly use
        // the context we have. Otherwise we want the application context, so we don't cling to the
        // activity.
        Context context;
        if (mUseDecorContext) {<!-- -->
            Context applicationContext = getContext().getApplicationContext();
            if (applicationContext == null) {<!-- -->
                context = getContext();
            } else {<!-- -->
                context = new DecorContext(applicationContext, getContext());
                if (mTheme != -1) {<!-- -->
                    context.setTheme(mTheme);
                }
            }
        } else {<!-- -->
            context = getContext();
        }
        return new DecorView(context, featureId, this, getAttributes());
    }

```

### PhoneWindow.generateLayout()

generateLayout()是创建contentView的方法，主要逻辑如下：
1. 通过getWindowStyle()获取到window的TypeArray，然后通过这个TypeArray来获取window相关的状态，从而来设置window的属性。（常见的状态如：是否有title，是否有actionbar等）1. 此处还有window.getContainer()的逻辑。举个例子，一个dialog会有自己的Window，Dialog的Window的Container就是 它的Activity的Window。1. 最终会通过id=ID_ANDROID_CONTENT获取到contentParent返回。ID_ANDROID_CONTENT = com.android.internal.R.id.content。（使用Fragement的时候经常会使用R.id.content，其实就是对应contentView）
```
    protected ViewGroup generateLayout(DecorView decor) {<!-- -->
        // Apply data from current theme.

        TypedArray a = getWindowStyle();

        if (false) {<!-- -->
            System.out.println("From style:");
            String s = "Attrs:";
            for (int i = 0; i &lt; R.styleable.Window.length; i++) {<!-- -->
                s = s + " " + Integer.toHexString(R.styleable.Window[i]) + "="
                        + a.getString(i);
            }
            System.out.println(s);
        }

        mIsFloating = a.getBoolean(R.styleable.Window_windowIsFloating, false);
        int flagsToUpdate = (FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR)
                &amp; (~getForcedWindowFlags());
        if (mIsFloating) {<!-- -->
            setLayout(WRAP_CONTENT, WRAP_CONTENT);
            setFlags(0, flagsToUpdate);
        } else {<!-- -->
            setFlags(FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR, flagsToUpdate);
        }

        if (a.getBoolean(R.styleable.Window_windowNoTitle, false)) {<!-- -->
            requestFeature(FEATURE_NO_TITLE);
        } else if (a.getBoolean(R.styleable.Window_windowActionBar, false)) {<!-- -->
            // Don't allow an action bar if there is no title.
            requestFeature(FEATURE_ACTION_BAR);
        }

        if (a.getBoolean(R.styleable.Window_windowActionBarOverlay, false)) {<!-- -->
            requestFeature(FEATURE_ACTION_BAR_OVERLAY);
        }

        if (a.getBoolean(R.styleable.Window_windowActionModeOverlay, false)) {<!-- -->
            requestFeature(FEATURE_ACTION_MODE_OVERLAY);
        }

        if (a.getBoolean(R.styleable.Window_windowSwipeToDismiss, false)) {<!-- -->
            requestFeature(FEATURE_SWIPE_TO_DISMISS);
        }

        if (a.getBoolean(R.styleable.Window_windowFullscreen, false)) {<!-- -->
            setFlags(FLAG_FULLSCREEN, FLAG_FULLSCREEN &amp; (~getForcedWindowFlags()));
        }

        if (a.getBoolean(R.styleable.Window_windowTranslucentStatus,
                false)) {<!-- -->
            setFlags(FLAG_TRANSLUCENT_STATUS, FLAG_TRANSLUCENT_STATUS
                    &amp; (~getForcedWindowFlags()));
        }

        if (a.getBoolean(R.styleable.Window_windowTranslucentNavigation,
                false)) {<!-- -->
            setFlags(FLAG_TRANSLUCENT_NAVIGATION, FLAG_TRANSLUCENT_NAVIGATION
                    &amp; (~getForcedWindowFlags()));
        }

        if (a.getBoolean(R.styleable.Window_windowOverscan, false)) {<!-- -->
            setFlags(FLAG_LAYOUT_IN_OVERSCAN, FLAG_LAYOUT_IN_OVERSCAN&amp;(~getForcedWindowFlags()));
        }

        if (a.getBoolean(R.styleable.Window_windowShowWallpaper, false)) {<!-- -->
            setFlags(FLAG_SHOW_WALLPAPER, FLAG_SHOW_WALLPAPER&amp;(~getForcedWindowFlags()));
        }

        if (a.getBoolean(R.styleable.Window_windowEnableSplitTouch,
                getContext().getApplicationInfo().targetSdkVersion
                        &gt;= android.os.Build.VERSION_CODES.HONEYCOMB)) {<!-- -->
            setFlags(FLAG_SPLIT_TOUCH, FLAG_SPLIT_TOUCH&amp;(~getForcedWindowFlags()));
        }

        a.getValue(R.styleable.Window_windowMinWidthMajor, mMinWidthMajor);
        a.getValue(R.styleable.Window_windowMinWidthMinor, mMinWidthMinor);
        if (DEBUG) Log.d(TAG, "Min width minor: " + mMinWidthMinor.coerceToString()
                + ", major: " + mMinWidthMajor.coerceToString());
        if (a.hasValue(R.styleable.Window_windowFixedWidthMajor)) {<!-- -->
            if (mFixedWidthMajor == null) mFixedWidthMajor = new TypedValue();
            a.getValue(R.styleable.Window_windowFixedWidthMajor,
                    mFixedWidthMajor);
        }
        if (a.hasValue(R.styleable.Window_windowFixedWidthMinor)) {<!-- -->
            if (mFixedWidthMinor == null) mFixedWidthMinor = new TypedValue();
            a.getValue(R.styleable.Window_windowFixedWidthMinor,
                    mFixedWidthMinor);
        }
        if (a.hasValue(R.styleable.Window_windowFixedHeightMajor)) {<!-- -->
            if (mFixedHeightMajor == null) mFixedHeightMajor = new TypedValue();
            a.getValue(R.styleable.Window_windowFixedHeightMajor,
                    mFixedHeightMajor);
        }
        if (a.hasValue(R.styleable.Window_windowFixedHeightMinor)) {<!-- -->
            if (mFixedHeightMinor == null) mFixedHeightMinor = new TypedValue();
            a.getValue(R.styleable.Window_windowFixedHeightMinor,
                    mFixedHeightMinor);
        }
        if (a.getBoolean(R.styleable.Window_windowContentTransitions, false)) {<!-- -->
            requestFeature(FEATURE_CONTENT_TRANSITIONS);
        }
        if (a.getBoolean(R.styleable.Window_windowActivityTransitions, false)) {<!-- -->
            requestFeature(FEATURE_ACTIVITY_TRANSITIONS);
        }

        mIsTranslucent = a.getBoolean(R.styleable.Window_windowIsTranslucent, false);

        final Context context = getContext();
        final int targetSdk = context.getApplicationInfo().targetSdkVersion;
        final boolean targetPreHoneycomb = targetSdk &lt; android.os.Build.VERSION_CODES.HONEYCOMB;
        final boolean targetPreIcs = targetSdk &lt; android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH;
        final boolean targetPreL = targetSdk &lt; android.os.Build.VERSION_CODES.LOLLIPOP;
        final boolean targetHcNeedsOptions = context.getResources().getBoolean(
                R.bool.target_honeycomb_needs_options_menu);
        final boolean noActionBar = !hasFeature(FEATURE_ACTION_BAR) || hasFeature(FEATURE_NO_TITLE);

        if (targetPreHoneycomb || (targetPreIcs &amp;&amp; targetHcNeedsOptions &amp;&amp; noActionBar)) {<!-- -->
            setNeedsMenuKey(WindowManager.LayoutParams.NEEDS_MENU_SET_TRUE);
        } else {<!-- -->
            setNeedsMenuKey(WindowManager.LayoutParams.NEEDS_MENU_SET_FALSE);
        }

        if (!mForcedStatusBarColor) {<!-- -->
            mStatusBarColor = a.getColor(R.styleable.Window_statusBarColor, 0xFF000000);
        }
        if (!mForcedNavigationBarColor) {<!-- -->
            mNavigationBarColor = a.getColor(R.styleable.Window_navigationBarColor, 0xFF000000);
            mNavigationBarDividerColor = a.getColor(R.styleable.Window_navigationBarDividerColor,
                    0x00000000);
        }

        WindowManager.LayoutParams params = getAttributes();

        // Non-floating windows on high end devices must put up decor beneath the system bars and
        // therefore must know about visibility changes of those.
        if (!mIsFloating) {<!-- -->
            if (!targetPreL &amp;&amp; a.getBoolean(
                    R.styleable.Window_windowDrawsSystemBarBackgrounds,
                    false)) {<!-- -->
                setFlags(FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS,
                        FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS &amp; ~getForcedWindowFlags());
            }
            if (mDecor.mForceWindowDrawsStatusBarBackground) {<!-- -->
                params.privateFlags |= PRIVATE_FLAG_FORCE_DRAW_STATUS_BAR_BACKGROUND;
            }
        }
        if (a.getBoolean(R.styleable.Window_windowLightStatusBar, false)) {<!-- -->
            decor.setSystemUiVisibility(
                    decor.getSystemUiVisibility() | View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
        }
        if (a.getBoolean(R.styleable.Window_windowLightNavigationBar, false)) {<!-- -->
            decor.setSystemUiVisibility(
                    decor.getSystemUiVisibility() | View.SYSTEM_UI_FLAG_LIGHT_NAVIGATION_BAR);
        }
        if (a.hasValue(R.styleable.Window_windowLayoutInDisplayCutoutMode)) {<!-- -->
            int mode = a.getInt(R.styleable.Window_windowLayoutInDisplayCutoutMode, -1);
            if (mode &lt; LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT
                    || mode &gt; LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER) {<!-- -->
                throw new UnsupportedOperationException("Unknown windowLayoutInDisplayCutoutMode: "
                        + a.getString(R.styleable.Window_windowLayoutInDisplayCutoutMode));
            }
            params.layoutInDisplayCutoutMode = mode;
        }

        if (mAlwaysReadCloseOnTouchAttr || getContext().getApplicationInfo().targetSdkVersion
                &gt;= android.os.Build.VERSION_CODES.HONEYCOMB) {<!-- -->
            if (a.getBoolean(
                    R.styleable.Window_windowCloseOnTouchOutside,
                    false)) {<!-- -->
                setCloseOnTouchOutsideIfNotSet(true);
            }
        }

        if (!hasSoftInputMode()) {<!-- -->
            params.softInputMode = a.getInt(
                    R.styleable.Window_windowSoftInputMode,
                    params.softInputMode);
        }

        if (a.getBoolean(R.styleable.Window_backgroundDimEnabled,
                mIsFloating)) {<!-- -->
            /* All dialogs should have the window dimmed */
            if ((getForcedWindowFlags()&amp;WindowManager.LayoutParams.FLAG_DIM_BEHIND) == 0) {<!-- -->
                params.flags |= WindowManager.LayoutParams.FLAG_DIM_BEHIND;
            }
            if (!haveDimAmount()) {<!-- -->
                params.dimAmount = a.getFloat(
                        android.R.styleable.Window_backgroundDimAmount, 0.5f);
            }
        }

        if (params.windowAnimations == 0) {<!-- -->
            params.windowAnimations = a.getResourceId(
                    R.styleable.Window_windowAnimationStyle, 0);
        }

        // The rest are only done if this window is not embedded; otherwise,
        // the values are inherited from our container.
        if (getContainer() == null) {<!-- -->
            if (mBackgroundDrawable == null) {<!-- -->
                if (mBackgroundResource == 0) {<!-- -->
                    mBackgroundResource = a.getResourceId(
                            R.styleable.Window_windowBackground, 0);
                }
                if (mFrameResource == 0) {<!-- -->
                    mFrameResource = a.getResourceId(R.styleable.Window_windowFrame, 0);
                }
                mBackgroundFallbackResource = a.getResourceId(
                        R.styleable.Window_windowBackgroundFallback, 0);
                if (false) {<!-- -->
                    System.out.println("Background: "
                            + Integer.toHexString(mBackgroundResource) + " Frame: "
                            + Integer.toHexString(mFrameResource));
                }
            }
            if (mLoadElevation) {<!-- -->
                mElevation = a.getDimension(R.styleable.Window_windowElevation, 0);
            }
            mClipToOutline = a.getBoolean(R.styleable.Window_windowClipToOutline, false);
            mTextColor = a.getColor(R.styleable.Window_textColor, Color.TRANSPARENT);
        }

        // Inflate the window decor.

        int layoutResource;
        int features = getLocalFeatures();
        // System.out.println("Features: 0x" + Integer.toHexString(features));
        if ((features &amp; (1 &lt;&lt; FEATURE_SWIPE_TO_DISMISS)) != 0) {<!-- -->
            layoutResource = R.layout.screen_swipe_dismiss;
            setCloseOnSwipeEnabled(true);
        } else if ((features &amp; ((1 &lt;&lt; FEATURE_LEFT_ICON) | (1 &lt;&lt; FEATURE_RIGHT_ICON))) != 0) {<!-- -->
            if (mIsFloating) {<!-- -->
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleIconsDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else {<!-- -->
                layoutResource = R.layout.screen_title_icons;
            }
            // XXX Remove this once action bar supports these features.
            removeFeature(FEATURE_ACTION_BAR);
            // System.out.println("Title Icons!");
        } else if ((features &amp; ((1 &lt;&lt; FEATURE_PROGRESS) | (1 &lt;&lt; FEATURE_INDETERMINATE_PROGRESS))) != 0
                &amp;&amp; (features &amp; (1 &lt;&lt; FEATURE_ACTION_BAR)) == 0) {<!-- -->
            // Special case for a window with only a progress bar (and title).
            // XXX Need to have a no-title version of embedded windows.
            layoutResource = R.layout.screen_progress;
            // System.out.println("Progress!");
        } else if ((features &amp; (1 &lt;&lt; FEATURE_CUSTOM_TITLE)) != 0) {<!-- -->
            // Special case for a window with a custom title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {<!-- -->
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogCustomTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else {<!-- -->
                layoutResource = R.layout.screen_custom_title;
            }
            // XXX Remove this once action bar supports these features.
            removeFeature(FEATURE_ACTION_BAR);
        } else if ((features &amp; (1 &lt;&lt; FEATURE_NO_TITLE)) == 0) {<!-- -->
            // If no other features and not embedded, only need a title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {<!-- -->
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else if ((features &amp; (1 &lt;&lt; FEATURE_ACTION_BAR)) != 0) {<!-- -->
                layoutResource = a.getResourceId(
                        R.styleable.Window_windowActionBarFullscreenDecorLayout,
                        R.layout.screen_action_bar);
            } else {<!-- -->
                layoutResource = R.layout.screen_title;
            }
            // System.out.println("Title!");
        } else if ((features &amp; (1 &lt;&lt; FEATURE_ACTION_MODE_OVERLAY)) != 0) {<!-- -->
            layoutResource = R.layout.screen_simple_overlay_action_mode;
        } else {<!-- -->
            // Embedded, so no decoration is needed.
            layoutResource = R.layout.screen_simple;
            // System.out.println("Simple!");
        }

        mDecor.startChanging();
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);

        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        if (contentParent == null) {<!-- -->
            throw new RuntimeException("Window couldn't find content container view");
        }

        if ((features &amp; (1 &lt;&lt; FEATURE_INDETERMINATE_PROGRESS)) != 0) {<!-- -->
            ProgressBar progress = getCircularProgressBar(false);
            if (progress != null) {<!-- -->
                progress.setIndeterminate(true);
            }
        }

        if ((features &amp; (1 &lt;&lt; FEATURE_SWIPE_TO_DISMISS)) != 0) {<!-- -->
            registerSwipeCallbacks(contentParent);
        }

        // Remaining setup -- of background and title -- that only applies
        // to top-level windows.
        if (getContainer() == null) {<!-- -->
            final Drawable background;
            if (mBackgroundResource != 0) {<!-- -->
                background = getContext().getDrawable(mBackgroundResource);
            } else {<!-- -->
                background = mBackgroundDrawable;
            }
            mDecor.setWindowBackground(background);

            final Drawable frame;
            if (mFrameResource != 0) {<!-- -->
                frame = getContext().getDrawable(mFrameResource);
            } else {<!-- -->
                frame = null;
            }
            mDecor.setWindowFrame(frame);

            mDecor.setElevation(mElevation);
            mDecor.setClipToOutline(mClipToOutline);

            if (mTitle != null) {<!-- -->
                setTitle(mTitle);
            }

            if (mTitleColor == 0) {<!-- -->
                mTitleColor = mTextColor;
            }
            setTitleColor(mTitleColor);
        }

        mDecor.finishChanging();

        return contentParent;
    }

```

# activity中window的初始化

window是在activity启动的时候被初始化的，widnow初始化的任务栈调用是：
- ActivityThread.performLaunchActivity()- Activity.attach()
Activity.attach()主要逻辑如下：
1. 创建PhoneWindow1. 为PhoneWindow设置windowManager，通过context.getSystemService()获取到windowManager。1. 当activity的mParent存在时，为window的container设置成mParent.getWindow()。（mParent和TabActivity有关，有兴趣的可以自行看下TabActivity源码）
```
    final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {<!-- -->
        attachBaseContext(context);

        mFragments.attachHost(null /*parent*/);

        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(this);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {<!-- -->
            mWindow.setSoftInputMode(info.softInputMode);
        }
        if (info.uiOptions != 0) {<!-- -->
            mWindow.setUiOptions(info.uiOptions);
        }
        mUiThread = Thread.currentThread();

        mMainThread = aThread;
        mInstrumentation = instr;
        mToken = token;
        mAssistToken = assistToken;
        mIdent = ident;
        mApplication = application;
        mIntent = intent;
        mReferrer = referrer;
        mComponent = intent.getComponent();
        mActivityInfo = info;
        mTitle = title;
        mParent = parent;
        mEmbeddedID = id;
        mLastNonConfigurationInstances = lastNonConfigurationInstances;
        if (voiceInteractor != null) {<!-- -->
            if (lastNonConfigurationInstances != null) {<!-- -->
                mVoiceInteractor = lastNonConfigurationInstances.voiceInteractor;
            } else {<!-- -->
                mVoiceInteractor = new VoiceInteractor(voiceInteractor, this, this,
                        Looper.myLooper());
            }
        }

        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags &amp; ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {<!-- -->
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager();
        mCurrentConfig = config;

        mWindow.setColorMode(info.colorMode);

        setAutofillOptions(application.getAutofillOptions());
        setContentCaptureOptions(application.getContentCaptureOptions());
    }

```