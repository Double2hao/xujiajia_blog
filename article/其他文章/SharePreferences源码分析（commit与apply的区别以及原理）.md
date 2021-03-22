#SharePreferences源码分析（commit与apply的区别以及原理）
# 前提概要

上一篇文章，笔者分析了SharedPreferencesImpl的原理，然而结尾有读者评论说想通过源码理解一下commit（）与apply（）的区别。由于上篇文章已经发布，就不特地加长篇幅了，在此新启一篇分析一下两者的区别。 如果对SharedPreferencesImpl的原理还是完全不了解的建议看一下上一篇文章，此篇文章主要只讲commit（）与apply（）的区别。

# 官方解释

近期Google Developers 中国网站已经正式发布，我们就去官网上看一下关于apply的解释。

>  
 <h2>apply</h2> 
 Added in API level 9 void apply () Commit your preferences changesback from this Editor to the SharedPreferences object it is editing.This atomically performs the requested modifications, replacing whatever is currently in the SharedPreferences. 
 Note that when two editors are modifying preferences at the same time,the last one to call apply wins. 
 Unlike commit(), which writes its preferences out to persistent storage synchronously, apply() commits its changes to the in-memory SharedPreferences immediately but starts an asynchronous commit to disk and you won’t be notified of any failures. If another editor on this SharedPreferences does a regular commit() while a apply() is still outstanding, the commit() will block until all async commits are completed as well as the commit itself. 
 As SharedPreferences instances are singletons within a process, it’s safe to replace any instance of commit() with apply() if you were already ignoring the return value. 
 You don’t need to worry about Android component lifecycles and their interaction with apply() writing to disk. The framework makes sure in-flight disk writes from apply() complete before switching states. 
 The SharedPreferences.Editor interface isn’t expected to be implemented directly. However, if you previously did implement it and are now getting errors about missing apply(), you can simply call commit() from apply(). 


通过上面，我们翻译一下，可以大概得到以下几点： 1、如果先后apply（）了几次，那么会以最后一次apply（）的为准。 2、commit（）是把内容同步提交到硬盘的。而apply（）先立即把修改提交到内存，然后开启一个异步的线程提交到硬盘，并且如果提交失败，你不会收到任何通知。 3、如果当一个apply（）的异步提交还在进行的时候，执行commit（）操作，那么commit（）是会阻塞的。而如果commit（）的时候，前面的commit（）还未结束，这个commit（）还是会阻塞的。（所以引起commit阻塞会有这两种原因） 4、由于SharePreferences在一个程序中的实例一般都是单例的，所以如果你不是很在意返回值的话，你使用apply（）代替commit（）是无所谓的。

# 源码分析

```
        public void apply() {
            final MemoryCommitResult mcr = commitToMemory();
            final Runnable awaitCommit = new Runnable() {
                    public void run() {
                        try {
                            mcr.writtenToDiskLatch.await();
                        } catch (InterruptedException ignored) {
                        }
                    }
                };

            QueuedWork.add(awaitCommit);

            Runnable postWriteRunnable = new Runnable() {
                    public void run() {
                        awaitCommit.run();
                        QueuedWork.remove(awaitCommit);
                    }
                };

            SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);

            // Okay to notify the listeners before it's hit disk
            // because the listeners should always get the same
            // SharedPreferences instance back, which has the
            // changes reflected in memory.
            notifyListeners(mcr);
        }

```

```
        public boolean commit() {
            MemoryCommitResult mcr = commitToMemory();
            SharedPreferencesImpl.this.enqueueDiskWrite(
                mcr, null /* sync write on this thread okay */);
            try {
                mcr.writtenToDiskLatch.await();
            } catch (InterruptedException e) {
                return false;
            }
            notifyListeners(mcr);
            return mcr.writeToDiskResult;
        }

```

代码其实差不多，主要有两点不同： 1、commit（）有返回值，apply（）没有返回值。正是验证了官方的解释：apply（）失败了是不会报错的。 2、有一行代码在commit（）中是直接执行的，而在apply（）中是放到了Runnable中，这行代码意思是等待文件写完：

```
mcr.writtenToDiskLatch.await();

```

为什么放到Runnable中，其实比较好推测，就是想放到线程池中执行呗，当然这仅仅是我们的推测，我们需要找到具体的代码证明。那么我们就查看调用这个Runnable的enqueueDiskWrite（）方法：

```
    private void enqueueDiskWrite(final MemoryCommitResult mcr,
                                  final Runnable postWriteRunnable) {
        final Runnable writeToDiskRunnable = new Runnable() {
                public void run() {
                    synchronized (mWritingToDiskLock) {
                        writeToFile(mcr);
                    }
                    synchronized (SharedPreferencesImpl.this) {
                        mDiskWritesInFlight--;
                    }
                    if (postWriteRunnable != null) {
                        postWriteRunnable.run();
                    }
                }
            };

        final boolean isFromSyncCommit = (postWriteRunnable == null);

        // Typical #commit() path with fewer allocations, doing a write on
        // the current thread.
        if (isFromSyncCommit) {
            boolean wasEmpty = false;
            synchronized (SharedPreferencesImpl.this) {
                wasEmpty = mDiskWritesInFlight == 1;
            }
            if (wasEmpty) {
                writeToDiskRunnable.run();
                return;
            }
        }

        QueuedWork.singleThreadExecutor().execute(writeToDiskRunnable);
    }

```

如上，我们可以看到writeToFile（）也就是写入文件的逻辑我们会放到writeToDiskRunnable 这个Runnable中，如果没有传递postWriteRunnable进来（也就是commit的情况），那么就会在当前线程执行写入文件操作，而如果传递了postWriteRunnable进来（也就是apply的情况），那么就会把写入文件的逻辑放到线程池中运行。 这里也是验证了官方的说明：apply（）写入文件的操作是异步的，而commit（）的写入文件的操作是在当前线程同步执行的。

>  
 关于写入文件操作的具体分析此处就不多说增加篇幅了，有兴趣的读者可以看上一篇文章  


# 总结

从源码来分析其实很简单，两者主要区别有两点： 1、commit（）有返回值，apply（）没有返回值。apply（）失败了是不会报错的。 2、apply（）写入文件的操作是异步的，会把Runnable放到线程池中执行，而commit（）的写入文件的操作是在当前线程同步执行的。 因此当两者都可以使用的时候还是推荐使用apply（），因为apply（）写入文件操作是异步执行的，不会占用主线程资源。