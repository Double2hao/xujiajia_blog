#线程池解析（二）——execute、addWorker源码解析
>  
 <h2>相关文章</h2> 
     


# 概述

线程池使用execute执行任务，在创建线程池后，开发者只要调用这个方法就能愉快的使用线程池了。 本文将探索下其内部实现逻辑。

# execute源码

```
    public void execute(Runnable command) {<!-- -->
        if (command == null)
            throw new NullPointerException();
        
        int c = ctl.get();
        if (workerCountOf(c) &lt; corePoolSize) {<!-- -->
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) &amp;&amp; workQueue.offer(command)) {<!-- -->
            int recheck = ctl.get();
            if (! isRunning(recheck) &amp;&amp; remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }

```

过程：
1. 如果正在工作的线程小于corePoolSize，那么会执行addWorker(command, true)。1. 如果线程已经等于或者超过corePoolSize，且线程池是Running状态，那么会将task加入到工作队列中。 (这里有个再次确认的逻辑，再次确认线程池是否在running的状态中，如果不是，会从队列中将这个task移除，并且执行拒绝逻辑。) 如果此时正在工作的线程是0，会执行addWorker(null, false)。1. 如果说线程池不在running状态，或者task添加到队列失败。那么会执行addWorker(command, false)。
# addWorker解析

execute的源码非常依赖于addWorker，接下来就看下addWorker的源码。

### addWorker源码

```
    private boolean addWorker(Runnable firstTask, boolean core) {<!-- -->
        retry:
        for (;;) {<!-- -->
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs &gt;= SHUTDOWN &amp;&amp;
                ! (rs == SHUTDOWN &amp;&amp;
                   firstTask == null &amp;&amp;
                   ! workQueue.isEmpty()))
                return false;

            for (;;) {<!-- -->
                int wc = workerCountOf(c);
                if (wc &gt;= CAPACITY ||
                    wc &gt;= (core ? corePoolSize : maximumPoolSize))
                    return false;
                if (compareAndIncrementWorkerCount(c))
                    break retry;
                c = ctl.get();  // Re-read ctl
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }

        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {<!-- -->
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {<!-- -->
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {<!-- -->
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());

                    if (rs &lt; SHUTDOWN ||
                        (rs == SHUTDOWN &amp;&amp; firstTask == null)) {<!-- -->
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        workers.add(w);
                        int s = workers.size();
                        if (s &gt; largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {<!-- -->
                    mainLock.unlock();
                }
                if (workerAdded) {<!-- -->
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {<!-- -->
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
    }

```

逻辑：
1. 确定下worker是否可以创建。线程池如果已经stop，或者处在shutdown状态但是已经到达corePoolSize，那么就会直接返回false。1. 通过CAS来增加线程个数，此处会根据参数中的core来判断是“创建核心线程”和“创建非核心线程”。创建和新线程个数不能大于corePoolSize，创建非核心线程个数不能大于maximumPoolSize。1. 创建Worker，并且将这个worker加入的HashSet中。这里也使用了ReentrantLock来保证线程池的状态。addWorker操作结束之后，就释放这个ReentrantLock。
### addWorker总结

由源码解析得，addWorker的几种调用场景如下：
1. addWorker(command, true) 当线程数小于corePoolSize时，创建核心线程并且运行task。1. addWorker(command, false) 当核心线程数已满，阻塞队列已满，并且线程数小于maximumPoolSize时，创建非核心线程并且运行task。1. addWorker(null, false) 如果工作线程为0是，创建一个核心线程但是不运行task。（主要是避免工作队列中还有任务，但是工作线程为0，导致工作队列中的任务一直没有执行）
# execute总结
1. 如果正在工作线程小于corePoolSize,创建核心线程并且运行task。1. 如果正在工作线程等于corePoolSize，会尝试将task加入工作队列。 (加入队列之后，会再次确认下状态，如果线程池已经关闭了，那么会移除这个任务并执行拒绝策略，如果所有的工作线程正好都运行完了，那么会再创建一个，避免工作队列中的任务没有线程去执行)1. 如果工作线程等于corePoolSize，并且工作队列已满。会创建非工作线程来执行这个任务，如果执行失败，即线程数已达maximumPoolSize，那么会执行拒绝策略。