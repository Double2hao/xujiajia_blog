#HandlerThread的使用以及原理
# HandlerThread

>  
 如果已经理解Handler,Loop,MessageQueue的工作原理看此篇文章会非常简单，若没有了解的读者，可以看下面的文章：  


首先我们先看一下官方的描述：

>  
 Handy class for starting a new thread that has a looper. The looper can then be used to create handler classes. Note that start() must still be called. 


大致意思是HandlerThread能够新建拥有Looper的线程。这个Looper能够用来新建其他的Handler。（线程中的Looper）需要注意的是，新建的时候需要被回调。

# 用法

一般情况下，我们会经常用Handler在子线程中更新UI线程，那是因为在主线程中有Looper循环，而HandlerThread新建拥有Looper的子线程又有什么用呢？ 必然是执行耗时操作。举个例子，数据实时更新，我们每10秒需要切换一下显示的数据，如果我们将这种长时间的反复调用操作放到UI线程中，虽说可以执行，但是这样的操作多了之后，很容易会让UI线程卡顿甚至崩溃。 于是，就必须在子线程中调用这些了。 HandlerThread继承自Thread，一般适应的场景，便是集Thread和Handler之所长，适用于会长时间在后台运行，并且间隔时间内（或适当情况下）会调用的情况，比如上面所说的实时更新。

>  
 其实使用HandlerThread的效果和使用Thread+Handler差不多。不过后者对开发者的要求更高。 


# 效果

功能:每2秒更新一下UI。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039377840.png" alt="这里写图片描述">

# 主要代码：

```
package com.example.double2.handlerthreadtest;

import android.os.Bundle;
import android.os.Handler;
import android.os.HandlerThread;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    private TextView tvMain;

    private HandlerThread mHandlerThread;
    //子线程中的handler
    private Handler mThreadHandler;
    //UI线程中的handler
    private Handler mMainHandler = new Handler();

    //以防退出界面后Handler还在执行
    private boolean isUpdateInfo;
    //用以表示该handler的常熟
    private static final int MSG_UPDATE_INFO = 0x110;

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvMain = (TextView) findViewById(R.id.tv_main);

        initThread();
    }


    private void initThread()
    {
        mHandlerThread = new HandlerThread("check-message-coming");
        mHandlerThread.start();

        mThreadHandler = new Handler(mHandlerThread.getLooper())
        {
            @Override
            public void handleMessage(Message msg)
            {
                update();//模拟数据更新

                if (isUpdateInfo)
                    mThreadHandler.sendEmptyMessage(MSG_UPDATE_INFO);
            }
        };

    }

    private void update()
    {
        try
        {
            //模拟耗时
            Thread.sleep(2000);
            mMainHandler.post(new Runnable()
            {
                @Override
                public void run()
                {
                    String result = "每隔2秒更新一下数据：";
                    result += Math.random();
                    tvMain.setText(result);
                }
            });

        } catch (InterruptedException e)
        {
            e.printStackTrace();
        }

    }

    @Override
    protected void onResume()
    {
        super.onResume();
        //开始查询
        isUpdateInfo = true;
        mThreadHandler.sendEmptyMessage(MSG_UPDATE_INFO);
    }

    @Override
    protected void onPause()
    {
        super.onPause();
        //停止查询
        //以防退出界面后Handler还在执行
        isUpdateInfo = false;
        mThreadHandler.removeMessages(MSG_UPDATE_INFO);
    }

    @Override
    protected void onDestroy()
    {
        super.onDestroy();
        //释放资源
        mHandlerThread.quit();
    }
}


```

# 原理：

HandlerThread源码如下：

```
public class HandlerThread extends Thread {
    int mPriority;
    int mTid = -1;
    Looper mLooper;

    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }
    
   
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }
    
   
    protected void onLooperPrepared() {
    }

    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
    
   
    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        
        // If the thread has been started, wait until the looper has been created.
        synchronized (this) {
            while (isAlive() &amp;&amp; mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

    
    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }

   
    public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

  
    public int getThreadId() {
        return mTid;
    }
}

```

首先我们可以看到HandlerThread继承自Thread，因此在run（）中的逻辑都是在子线程中运行的。

接下来就是两个关键的方法，run（）和getLooper（）： run（）中可以看到是很简单的创建Looper以及让Looper工作的逻辑。 run()里面当mLooper创建完成后有个notifyAll()，getLooper()中有个wait()，这有什么用呢？因为的mLooper在一个线程中执行创建，而我们的handler是在UI线程中调用getLooper（）初始化的。 也就是说，我们必须等到mLooper创建完成，才能正确的返回。getLooper();wait(),notify()就是为了解决这两个线程的同步问题。