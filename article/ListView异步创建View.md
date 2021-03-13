#ListView异步创建View
# 前提概要

异步创建View这种操作一般情况下是用不到的，包括笔者之前自学阶段也是闻所未闻。  这定然是突破了我们一般编程的思维——UI操作难道不是只能在UI线程中吗？  是的，UI操作只能在UI线程中，但是UI控件的操作却是可以异步执行的。

考虑一下以下需求：

>  
 我们要展示一个ListView，ListView中的数据和布局都是我们网络获取的，我们预先并不知道。 


以往的我们使用一个Listview一般都是为了展示一类布局相同的信息，这种情况下，我们可以通过adapter的getView（）方法中的convertView来实现View的复用，使View不用反复创建。  比如以下：  <img src="https://img-blog.csdn.net/20170714173006847?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述" title="">  但是倘若ListView的每个Item布局都不相同，并且布局可能是网络动态获取的，我们并不能预先得知。这种情况就不能使用convertView了，那要怎么处理呢。笔者使用的便是异步创建View的方式。

# 异步创建View

## 主要流程：

1、既然是异步创建，那么创建线程池当然是必要的，倘若使用new Thread（）的方式，每次创建线程的消耗都超过创建View的消耗了。  2、如需求所说，我们预先并不知道我们会加载什么布局，什么控件，所以我们也不知道异步创建什么。所以在第一次创建View的时候，是在主线程创建的，然后放入到回收池中，回收池接受到了这个View，通过getClass（）的方式就知道自己需要异步创建什么View了。  3、存在多线程对同一个变量进行存取操作，就必然涉及到锁的问题。比如：第一次获取到View，回收池接受到之后开始在池中创建它的副本，但是副本还未创建完成（创建操作是异步的），adapter就进行了取的操作，这必然是会有问题的。这里笔者使用乐观锁的方式，给获取操作加上try-catch，尽量去相信这种问题不会发生，如果发生了就直接返回null，在主线程创建。

## 各个类的主要逻辑

<img src="https://img-blog.csdn.net/20170714171305965?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述" title="">

**CreateThreadPoolExecutor**：  线程池，每当有线程需要创建View的时候，就将任务交给他执行。  **HalControlListAdapter**：  继承自BaseAdapter。在getView（）中获取View的时候首先去HalControlListviewPool中获取，如果没有获取到，就主线程自己创建。一般情况下，如果是主线程创建的，说明是第一次创建，所以要放入回收池回收，以便让回收池创建副本。  **HalControlListviewPool**：  使用HashMap和LinkedList的数据格式来回收，存取View。  如果有View传进来，就为这个View创建arraySize个副本，以供后面的View获取。（在demo中 arraySize=1）

# 示例（源码在文章结尾）

每个Item随机创建TextView或者EditView，如果在主线程创建的，就设置Text“主线程xxx”，如果是异步创建的，就不设置text，如下：  <img src="https://img-blog.csdn.net/20170714172436665?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述" title="">  笔者为了演示，设置在回收池中每个View的副本只创建一个（arraySize=1），如果设置较多，基本不会再主线程创建View。  当然，即使是只有一个副本，可以看到，大部分的View还是异步创建的。  这样就能节省很多主线程的资源。

# 代码

## CreateThreadPoolExecutor

```
package com.example.xujiajia_sx.asynccreateui;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class CreateThreadPoolExecutor extends ThreadPoolExecutor {<!-- -->
    private static CreateThreadPoolExecutor createThreadPoolExecutor;

    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    //核心线程数
    private static final int CORE_POOL_SIZE = CPU_COUNT * 4 + 1;
    //允许创建的最大线程数
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 5 + 1;
    //当前线程池线程综述大于核心线程数时，种植多余的空闲线程的时间
    private static final long KEEP_ALIVE = 10L;

    //用于给线程池创建线程的线程工厂类
    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);

        @Override
        public Thread newThread(Runnable r) {
            return new Thread(r, "listview#" + mCount.getAndIncrement());
        }
    };

    private CreateThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                                     TimeUnit unit, BlockingQueue&lt;Runnable&gt; workQueue, ThreadFactory threadFactory) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory);
    }

    public static synchronized CreateThreadPoolExecutor getInstance() {
        if (createThreadPoolExecutor == null) {
            createThreadPoolExecutor = new CreateThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE,
                    KEEP_ALIVE, TimeUnit.SECONDS, new LinkedBlockingDeque&lt;Runnable&gt;(), sThreadFactory);
        }
        return createThreadPoolExecutor;
    }
}

```

## HalControlListAdapter

```
package com.example.xujiajia_sx.asynccreateui;

import android.content.Context;
import android.util.Log;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

import java.util.Random;


/**
 * Created by breakerror on 2017/6/12.
 */

public class HalControlListAdapter extends BaseAdapter {<!-- -->
    private Context mContext = null;
    private HalControlListviewPool mPool;

    public HalControlListAdapter(Context context) {
        mContext = context;
        mPool = new HalControlListviewPool(mContext);
    }

    @Override
    public int getCount() {
        return 100;
    }

    @Override
    public Object getItem(int position) {
        return null;
    }

    @Override
    public long getItemId(int position) {
        return 0;
    }

    int count = 0;

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        int key = new Random().nextInt(2);

        View view = mPool.get((long) key);
        if (view == null) {
            view = createView(key, position);
            mPool.recycle(key, view);
        }

        return view;
    }

    public View createView(int key, int position) {
        View view;
        switch (key) {
            case 0:
                view = new TextView(mContext);
                ((TextView) view).setText("TextView主线程" + position);
                break;
            case 1:
            default:
                view = new EditText(mContext);
                ((EditText) view).setText("EditText主线程" + position);
                break;

        }
        return view;
    }


}

```

## HalControlListviewPool

```
package com.example.xujiajia_sx.asynccreateui;

import android.content.Context;
import android.view.View;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.util.HashMap;
import java.util.LinkedList;
import java.util.concurrent.ThreadPoolExecutor;

/**
 * Created by breakerror on 2017/6/19.
 */

public class HalControlListviewPool {<!-- -->
    private Context mContext = null;
    private HashMap&lt;Long, LinkedList&lt;View&gt;&gt; mViewPool = null;
    //每个对象异步创建的个数
    private final int arraySize = 1;
    //创建一个静态的线程池对象
    private static ThreadPoolExecutor THREAD_POOL_EXECUTOR = null;

    public HalControlListviewPool(Context context) {
        mViewPool = new HashMap&lt;&gt;();
        mContext = context;
        THREAD_POOL_EXECUTOR = CreateThreadPoolExecutor.getInstance();
    }

    public boolean recycle(long t, View view) {
        LinkedList&lt;View&gt; subView = mViewPool.get(t);
        if (null == subView) {
            mViewPool.put(t, new LinkedList&lt;View&gt;());
            THREAD_POOL_EXECUTOR.execute(new createTask(view.getClass(), t));
            return true;
        } else {
            return false;
        }
    }

    public View get(long t) {
        LinkedList&lt;View&gt; subView = mViewPool.get(t);
        View view = null;
        if (null != subView &amp;&amp; subView.size() &gt; 0) {
            //如果子线程只写了一半就读取会出错，如果子线程没写完就返回null
            try {
                view = subView.getFirst();
                subView.removeFirst();
                THREAD_POOL_EXECUTOR.execute(new createTask(view.getClass(), t));
            } catch (Exception e) {
            }
        }
        return view;
    }


    public class createTask implements Runnable {<!-- -->

        private Class mClass;
        private long mT;

        public createTask(Class c, long t) {
            mClass = c;
            mT = t;
        }

        @Override
        public void run() {
            try {
                Constructor c = mClass.getConstructor(Context.class);
                LinkedList&lt;View&gt; subView = mViewPool.get(mT);
                while (subView.size() &lt; arraySize) {
                    subView.add((View) c.newInstance(mContext));
                }
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (NoSuchMethodException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
        }
    }

}

```

## MainActivity

```
package com.example.xujiajia_sx.asynccreateui;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.widget.LinearLayoutCompat;
import android.widget.LinearLayout;
import android.widget.ListView;

public class MainActivity extends AppCompatActivity {<!-- -->

    private ListView lvMain;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        lvMain=(ListView)findViewById(R.id.lv_main);

    }

    @Override
    protected void onStart() {
        super.onStart();
        lvMain.setAdapter(new HalControlListAdapter(this));
    }
}

```

## 源码下载地址

