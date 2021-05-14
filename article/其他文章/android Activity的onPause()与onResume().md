#android Activity的onPause()与onResume()
**官方文档地址：**

 

**Pause和Resume一个Activity**

在一般的app使用中，前台的activity一般是会被视觉组件所遮住的，这就会导致activity的pause。举个例子，当一个半透明的activity打开的时候（就像某种形式的对话框一样），这之前的activity会pause。只要activity仍然部分可见，但是当下activity并不可操作，它就处在pause中了。

但是，一旦这个activity全部被遮住了，并且不可见，它就stop了。（这会在下节课讨论到）

当你的activity进入pause状态的时候，这个系统调用了在activity中的onPause()这个方法，onPause ()这个方法让你能够结束一些正在进行的任务，而这些任务在停止的时候就不能继续了（比如一个视频）。它也能够让你在用户执意要离开你的app时，留存应该被永久保存的信息。如果用户从pause的状态又回到了你的activity，这个系统resume这个activity并且调用了onResume()这个方法。

 

注意：当你的activity收到了一个调用onPause()的请求，它可能表示这个activity将会被停止一段时间并且使用者很可能会再次回到你的activity来。但是这也很可能表示着用户正在离开你的app。

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039791030.png">

图片：当一个半透明的activity遮住了你的activity，这个系统调用了onPause()，并且activity在pause的状态中等待。如果用户当它仍然pause的时候再次回到了activity，这个系统会调用onResume()。

 

**pause你的Activity**

当这个系统为你的activity调用onPause()的时候，讲道理的话，它意味着你的activity仍然是部分可见的，但是也非常有可能表示是用户正在离开你的activity并且它之后将很快进入stop的状态。你应该经常会在以下情况下用到onPause()。

1、  结束占用CPU的动画或者其他正在运行任务。

2、  提交没有保存的改变，但是只有在用户离开的时候希望这些改变被保存的时候。

3、  释放系统资源，就像广播接收者，对传感器的操纵（就像GPS），或者在acitivity被pause之后和用户不需要的时候，任何可能影响你电池寿命的资源。

 

举个例子，如果你的应用用到了camera，那么onpause()方法是一个很好的释放它的方法。

 

```
@Override
public void onPause() {
    super.onPause();  // Always call the superclass method first

    // Release the Camera because we don't need it when paused
    // and other activities might need to use it.
    if (mCamera != null) {
        mCamera.release()
        mCamera = null;
    }
}
```

 

 

 

总体上说，你不应该使用onPause()来保存用户的改变到永久的存储空间中（比如表单输入个人信息）。唯一的你需要用onPause()来把用户的改变存储到永久内存中的情况是，当你的某县用户需要自动化保存的时候（比如写一封email）。但是在onPause()中你应该避免执行占用大量CPU的工作，比如写入数据库。因为这会减慢你可视化转换到下一个activity的速度。（你应该把这种繁重的关闭操作放到onstop()中）

在onPause()方法中，你应该要保持一部分的操作，来保证你的activity结束的时候能够以较快的速度跳转到下一个用户目的地。

 

注意：当你的activity被pause的时候，这个activity实例在内存中是占用位置的，而且在Activity被resume的时候会被再次唤醒。你不需要在resume状态之前的任何回调函数中，重新初始化任何已经被创建过的组件。

 

**resume你的Activity**

当用户从pause的状态resume你的activity的时候，系统调用了onResume()的函数。

 

考虑到系统每次进入前台运行的时候都会调用这个函数，即使是在这个activity已经被创建过的情况下。同样的你应该实现onResume()来初始化你在onPause()中释放的组件，并且并且执行一些其他在当activity进入resume状态的时候需要执行的初始化（比如打开动画，初始化只有activity获取了焦点后才需要使用的组件）。

 

接下来的onResume()的例子是与onPause()在上面的例子相配的，所以它初始化了在activity被pause的时候需要释放的camera。

 

```
@Override
public void onResume() {
    super.onResume();  // Always call the superclass method first

    // Get the Camera instance as the activity achieves full user focus
    if (mCamera == null) {
        initializeCamera(); // Local method to handle camera init
    }
}
```

 

 

 

 

 