#ViewTreeObserver 监听View的状态
>  
 官方文档 https://developer.android.com/reference/android/view/ViewTreeObserverv 


# 概述

近期碰到要再OnCreate中获取View绘制结束之后的宽高，用到了ViewTreeObserver.OnGlobalLayoutListener。

鉴于ViewTreeObserver的确比较实用，于是写本文记录下。

# 官方描述

>  
 A view tree observer is used to register listeners that can be notified of global changes in the view tree. Such global events include, but are not limited to, layout of the whole tree, beginning of the drawing pass, touch mode change… A ViewTreeObserver should never be instantiated by applications as it is provided by the views hierarchy. 


### 个人翻译

>  
 ViewTreeObserver可以用来注册一些监听者，这些监听者用于监听view Tree中一些全局的变化。类似的一些全局事件有：整个View tree的布局，开始绘制事件，触摸事件的变化等。鉴于ViewTreeObserver是由View hierarchy提供的，因此永远不应该由开发者来创建。 


# 重要接口

### ViewTreeObserver.OnDrawListener

当View开始绘制的时候，可以用这个接口来定义回调。

### ViewTreeObserver.OnGlobalFocusChangeListener

当View的焦点状态改变的时候，可以用这个接口来定义回调。

### ViewTreeObserver.OnGlobalLayoutListener

当View的layout状态或者可见状态变化的时候，可以用这个接口来定义回调。

### ViewTreeObserver.OnPreDrawListener

当View将要开始绘制的时候，可以用这个接口来定义回调。

### ViewTreeObserver.OnScrollChangedListener

当View中的内容开始滚动的时候，可以用这个接口来定义回调。

### ViewTreeObserver.OnTouchModeChangeListener

当View的触摸状态变化的时候，可以用这个接口来定义回调。

### ViewTreeObserver.OnWindowAttachListener

当View的层级被 放到window上 或者 从window移除 的时候，可以用这个接口来定义回调。

### ViewTreeObserver.OnWindowFocusChangeListener

当View的层级的window的焦点状态不变化的时候，可以用这个接口来定义回调。