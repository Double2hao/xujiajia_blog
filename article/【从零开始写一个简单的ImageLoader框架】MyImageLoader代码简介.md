#【从零开始写一个简单的ImageLoader框架】MyImageLoader代码简介
# 相关文章







# 前提概要

笔者仅仅对各个java文件作用以及一些关键方法进行阐述，其余很多需要掌握的知识点已于上一篇文章中给出相应链接，还需读者自己学习。

# 代码预览（须结合源码看）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2220.png" alt="这里写图片描述">

demo包中的文件都用于演示，也不多加阐述了。 主要为imageLoader中的内容。

# BitmapUtils

此文件中包含了**图片压缩**的公共方法。

# ImageDiskLruCache

硬盘缓存。

# ImageLruCache

内存缓存。

# ImageThreadPoolExecutor

线程池。

# LoadBitmapTask

```
public class LoadBitmapTask implements Runnable 

```

如上图，LoadBitmapTask为一个Runnable，用于放入线程池中运行。主要逻辑为，从磁盘或者网络获取bitmap，之后再imageView中显示，如下：

```
@Override
    public void run() {
        //从磁盘或者网络获取bitmap
        final Bitmap bitmap = loadBitmap(uri, reqWidth, reqHeight);

        //使用handler更新UI
        TaskResult loaderResult = new TaskResult(imageView, bitmap);
        imageHandler.obtainMessage(MESSAGE_POST_RESULT, loaderResult).sendToTarget();

        if (callback != null) {
            Handler handler = new Handler(Looper.getMainLooper());
            handler.post(new Runnable() {
                @Override
                public void run() {
                    callback.onResponse(bitmap);
                }
            });

        }
    }

```

# MD5Utils

将url通过MD5转化为唯一的字符串，用于标识。主要用于内存缓存和磁盘缓存。

# MyImageLoader

暴露给使用者的公共方法类，主要方法为bindBitmap()，如下: 首先从内存缓存中获取bitmap，如果内存缓存中没有，就在磁盘缓存中查找，如果磁盘缓存中也没有，就从网上获取，并且缓存到磁盘缓存中。

```
public void bindBitmap(final String uri, final ImageView imageView, final int reqWidth, final int reqHeight) {
        //设置加载loadding图片
        imageView.setImageResource(R.drawable.ic_loading);

        //从内存缓存中获取bitmap
        Bitmap bitmap = imageLrucache.loadBitmapFromMemCache(uri);
        if (bitmap != null) {
            imageView.setImageBitmap(bitmap);
            return;
        }
        //磁盘缓存包含在LoadBitmapTask中
        LoadBitmapTask loadBitmapTask = new LoadBitmapTask(mContext, imageView, uri, reqWidth, reqHeight);
        //使用线程池去执行Runnable对象
        THREAD_POOL_EXECUTOR.execute(loadBitmapTask);

    }

```

# MyUtils

工具类，包含Log，Toast等公共方法。

# NetWork

包含网络的公共方法，主要通过url获取到图片。

# TaskResult

用于Handler的Message的传递,代码如下。

```
public class TaskResult {
    public ImageView imageView;
    public Bitmap bitmap;

    public TaskResult(ImageView imageView, Bitmap bitmap) {
        this.imageView = imageView;
        this.bitmap = bitmap;

    }


```

```
//使用handler更新UI
        TaskResult loaderResult = new TaskResult(imageView, bitmap);
        imageHandler.obtainMessage(MESSAGE_POST_RESULT, loaderResult).sendToTarget();


```

```
//对ImageView的操作属于对UI的操作，不能再子线程中进行，所以要用到Handler
    private Handler imageHandler = new Handler(Looper.getMainLooper()) {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            if (msg.what == MESSAGE_POST_RESULT) {
                //给imageView加载bitmap
                TaskResult result = (TaskResult) msg.obj;
                ImageView imageView = result.imageView;

                if (result.bitmap != null) {
                    imageView.setImageBitmap(result.bitmap);
                } else {
                    imageView.setImageResource(R.drawable.ic_error);
                }
            }
        }
    };

```

# 项目下载地址

