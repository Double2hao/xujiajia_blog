#【从零开始写一个简单的ImageLoader框架】ImageLoader分析
# 相关文章







#项目涉及知识点

## 1、网络获取

## 2、图片压缩

## 3、内存缓存

## 4、磁盘缓存

## 5、线程池

## 6、Handler

## 7、RecyclerView（demo中使用）

# ImageLoader获取图片一般流程

1、从**内存缓存**中查看是否有该图片，如果有就设置到imageView上。

2、如果没有**内存缓存**，就从**磁盘缓存**中查看是否有该图片，如果有就设置到imageView上,并且把该图片设置到**内存缓存**中。

3、如果既没有**内存缓存**也没有**磁盘缓存**，那么就直接从网络获取该图片，并使用**磁盘缓存**，然后从磁盘中读取该图片。将图片根据ImageView的宽高压缩后进行显示。

4、由于获取**内存缓存**较快，所以不需要使用线程。 而**磁盘缓存**由于存在从网络下载图片这一较为耗时的环节，所以要使用线程。 考虑到可能会有多个图片同时显示，所以就要使用**线程池**进行统一管理。

5、在线程中无法直接更新UI，所以也要使用到**Handler**。

# 从代码了解流程(建议结合源码看)

## 代码预览

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2060.png" alt="这里写图片描述">

在笔者demo中主要使用到的方法为bindBitMap，此方法使用与ImageRecyclerAdapter中

```
public void setData(String url) {
            //垂直线性布局
            //加载高清图片并按宽高压缩
            //MyUtils.Log(url);
            MyImageLoader.getInstance(context).bindBitmap(url, imageView, width, height);

        }

```

我们直接跳转到bindBitmap中看逻辑：

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

可以看到，如果内存缓存读到了bitmap，那么就会直接显示并且退出，如果没有，那么就会执行LoadBitmapTask，LoadBitmapTask是一个Runnable对象,使用线程池调用时，它的逻辑会是在线程中进行的，我们可以继续看一下里面的逻辑。

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

我们发现loadBitmap(uri, reqWidth, reqHeight)在之后，就直接使用Handler进行UI的操作了，那么线程逻辑必然在这个里面。

```
 private Bitmap loadBitmap(String uri, int reqWidth, int reqHeight) {
        Bitmap bitmap = null;
        try {
            //从磁盘缓存中获取bitmap
            bitmap = MyImageLoader.getImageDiskLruCache(mContext).loadBitmapFromDiskCache(uri, reqWidth, reqHeight);
            if (bitmap != null) {
                MyUtils.Log("从磁盘缓存中获取到了bitmap");
                //添加到内存缓存中
                MyImageLoader.getImageLruCache().put(MD5Utils.hashKeyFromUrl(uri), bitmap);
                return bitmap;
            } else {
                //从网络下载bitmap到磁盘缓存，并从磁盘缓存中获取bitmap
                bitmap = loadBitmapFromHttp(uri, reqWidth, reqHeight);
                if (bitmap != null) {
                    MyUtils.Log("从网络下载并保存到磁盘并从中读取bitmap成功");
                }

            }

        } catch (IOException e) {
            e.printStackTrace();
        }
        if (bitmap == null &amp;&amp; !mIsDiskLruCacheCreated) {
            //如果sd卡已满，无法使用磁盘缓存，则通过网络下载bitmap，一般不会调用这一步
            bitmap = NetWork.downloadBitmapFromUrl(uri);
            MyUtils.Log("sd卡满了，直接从网络获取");
        }
        return bitmap;
    }

```

如上，我们就找到了磁盘缓存的代码了：从磁盘缓存中如果找到了图片，就会将图片添加到内存缓存中并返回bitmap，如果没有找到，就会从网上进行下载。

# 知识点详述链接

以下知识点于《Android开发艺术探索》中均有阐述。

## 1、图片压缩

此文章来自 中文版的Android Training。 

## 2、内存缓存

此文章来自 中文版的Android Training。 

## 3、磁盘缓存

Android DiskLruCache完全解析，硬盘缓存的最佳方案 此文章为郭霖前辈所著。 

## 4、线程池

此文章来自 中文版的Android Training。 

## 5、Handler

Handler类和Handler,Loop,MessageQueue的工作原理 

## 7、RecyclerView（demo中使用）

android RecyclerView布局真的只是那么简单！(笔者原创) 

# 项目下载地址

