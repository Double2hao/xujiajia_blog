#线程池解析（三）——Worker源码解析
>  
 <h2>相关文章</h2> 
     


# 概述

在ThreadPoolExecutor中以Worker为单位对工作线程进行管理。 那么Worker具体是做了什么呢？本文将围绕这个话题展开。

# Worker源码

```
   private final class Worker
       extends AbstractQueuedSynchronizer
       implements Runnable
   {<!-- -->
       /**
        * This class will never be serialized, but we provide a
        * serialVersionUID to suppress a javac warning.
        */
       private static final long serialVersionUID = 6138294804551838833L;

       /** Thread this worker is running in.  Null if factory fails. */
       final Thread thread;
       /** Initial task to run.  Possibly null. */
       Runnable firstTask;
       /** Per-thread task counter */
       volatile long completedTasks;

       /**
        * Creates with given first task and thread from ThreadFactory.
        * @param firstTask the first task (null if none)
        */
       Worker(Runnable firstTask) {<!-- -->
           setState(-1); // inhibit interrupts until runWorker
           this.firstTask = firstTask;
           this.thread = getThreadFactory().newThread(this);
       }

       /** Delegates main run loop to outer runWorker. */
       public void run() {<!-- -->
           runWorker(this);
       }

       // Lock methods
       //
       // The value 0 represents the unlocked state.
       // The value 1 represents the locked state.

       protected boolean isHeldExclusively() {<!-- -->
           return getState() != 0;
       }

       protected boolean tryAcquire(int unused) {<!-- -->
           if (compareAndSetState(0, 1)) {<!-- -->
               setExclusiveOwnerThread(Thread.currentThread());
               return true;
           }
           return false;
       }

       protected boolean tryRelease(int unused) {<!-- -->
           setExclusiveOwnerThread(null);
           setState(0);
           return true;
       }

       public void lock()        {<!-- --> acquire(1); }
       public boolean tryLock()  {<!-- --> return tryAcquire(1); }
       public void unlock()      {<!-- --> release(1); }
       public boolean isLocked() {<!-- --> return isHeldExclusively(); }

       void interruptIfStarted() {<!-- -->
           Thread t;
           if (getState() &gt;= 0 &amp;&amp; (t = thread) != null &amp;&amp; !t.isInterrupted()) {<!-- -->
               try {<!-- -->
                   t.interrupt();
               } catch (SecurityException ignore) {<!-- -->
               }
           }
       }
   }

```

# AbstractQueuedSynchronizer

### AQS作用

>  
 如果对AQS不是很了解的读者可以看下笔者前面的文章：  


Worker继承了AbstractQueuedSynchronizer，主要目的有两个：
1. 将锁的粒度细化到每个工Worker。
>  
 如果多个Worker使用同一个锁，那么一个Worker Running持有锁的时候，其他Worker就无法执行，这显然是不合理的。 

1. 直接使用CAS获取，避免阻塞。
>  
 如果这个锁使用阻塞获取，那么在多Worker的情况下执行shutDown。如果这个Worker此时正在Running无法获取到锁，那么执行shutDown()线程就会阻塞住了，显然是不合理的。 


### 源码解析

```
        // Lock methods
        //
        // The value 0 represents the unlocked state.
        // The value 1 represents the locked state.

        protected boolean isHeldExclusively() {<!-- -->
            return getState() != 0;
        }

        protected boolean tryAcquire(int unused) {<!-- -->
            if (compareAndSetState(0, 1)) {<!-- -->
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        protected boolean tryRelease(int unused) {<!-- -->
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        public void lock()        {<!-- --> acquire(1); }
        public boolean tryLock()  {<!-- --> return tryAcquire(1); }
        public void unlock()      {<!-- --> release(1); }
        public boolean isLocked() {<!-- --> return isHeldExclusively(); }

```

从源码得知，Worker中是通过CAS去获取锁的，期间不会有阻塞的逻辑。
- 如果获取成功。会记录下当前拿到锁的线程，然后返回 true。- 如果获取失败。会直接返回false.
### AQS使用(interruptIdleWorkers源码)

直接讲源码可能还是比较难以理解，我们直接从interruptIdleWorkers的源码来理解下Worker中的AQS是如何使用的。

```
    private void interruptIdleWorkers(boolean onlyOne) {<!-- -->
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {<!-- -->
            for (Worker w : workers) {<!-- -->
                Thread t = w.thread;
                if (!t.isInterrupted() &amp;&amp; w.tryLock()) {<!-- -->
                    try {<!-- -->
                        t.interrupt();
                    } catch (SecurityException ignore) {<!-- -->
                    } finally {<!-- -->
                        w.unlock();
                    }
                }
                if (onlyOne)
                    break;
            }
        } finally {<!-- -->
            mainLock.unlock();
        }
    }

```

interruptIdleWorkers是用来关闭空闲的Worker的，其逻辑主要如下：
1. 获取到线程池的ReentrantLock，上锁，执行完操作后再解锁。1. 遍历Worker，尝试获取worker的锁，如果可以获取到就说明是空闲的，interrupt这个Worker的线程。(如果一个Worker不是空闲的，那么它的锁会被占用，CAS会获取失败)
# Worker.run() 与 runWorker()

另外Worker还实现了Runnable，run()方法中最终是走到了线程池的runWorker()方法。源码如下：

```
        public void run() {<!-- -->
            runWorker(this);
        }

```

```
    final void runWorker(Worker w) {<!-- -->
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {<!-- -->
            while (task != null || (task = getTask()) != null) {<!-- -->
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &amp;&amp;
                      runStateAtLeast(ctl.get(), STOP))) &amp;&amp;
                    !wt.isInterrupted())
                    wt.interrupt();
                try {<!-- -->
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {<!-- -->
                        task.run();
                    } catch (RuntimeException x) {<!-- -->
                        thrown = x; throw x;
                    } catch (Error x) {<!-- -->
                        thrown = x; throw x;
                    } catch (Throwable x) {<!-- -->
                        thrown = x; throw new Error(x);
                    } finally {<!-- -->
                        afterExecute(task, thrown);
                    }
                } finally {<!-- -->
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {<!-- -->
            processWorkerExit(w, completedAbruptly);
        }
    }

```

逻辑如下：
1. 首先会去Worker的firstTask中取任务，如果没有就使用getTask()方法去取。（getTask()是会阻塞的,解析在后文）取到任务之后就会占用Worker的锁。1. 如果线程池目前状态大于或等于STOP（STOP,TIDYING,TERMINATED），那么需要把当前Worker中的线程也中止。（等待队列中的任务也不会再执行了）1. 如果线程池目前没有关闭。此时会执行task.run()，在当前线程执行任务。1. 此任务的逻辑执行完毕后，会释放Worker的锁。
### getTask()源码解析

```
    private Runnable getTask() {<!-- -->
        boolean timedOut = false; // Did the last poll() time out?

        for (;;) {<!-- -->
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs &gt;= SHUTDOWN &amp;&amp; (rs &gt;= STOP || workQueue.isEmpty())) {<!-- -->
                decrementWorkerCount();
                return null;
            }

            int wc = workerCountOf(c);

            // Are workers subject to culling?
            boolean timed = allowCoreThreadTimeOut || wc &gt; corePoolSize;

            if ((wc &gt; maximumPoolSize || (timed &amp;&amp; timedOut))
                &amp;&amp; (wc &gt; 1 || workQueue.isEmpty())) {<!-- -->
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }

            try {<!-- -->
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {<!-- -->
                timedOut = false;
            }
        }
    }

```

逻辑如下：
1. 如果“线程池已经处于shutdown并且线程池为空”或者“线程池已经大于等于stop”，那么此时会递减Worker的数量并且直接返回。
>  
 - SHUTDOWN 不会接受新的任务，但是会执行队列中的任务。- STOP 不会接受新的任务，也不会执行队列中的任务。 

1. 如果此时Worker的数量多于maximumPoolSize。或者Worker数量大于corePoolSize并且存在等待超时的情况，那么会去尝试减少Worker的数量。（根据超时逻辑来回收Worker）1. 阻塞等待去获取workQueue中的任务，获取到就返回，同时会存在超时逻辑。
>  
 超时逻辑： 核心线程数需要设置allowCoreThreadTimeOut才会超时，否则一直不会回收。 非核心线程一直有超时逻辑，超时不用就会被回收。 


# 总结

### getTask总结
- getTask中比较中比较重要的功能是超时回收逻辑。 核心线程数需要设置allowCoreThreadTimeOut才会超时，否则一直不会回收。 非核心线程一直有超时逻辑，超时不用就会被回收。- getTask也支持了线程池的shutDown与stop状态。 如果处于shutDown状态并且任务队列为空，会回收Worker。 如果处于Stop状态，不管任务队列什么样子，会回收Worker。
### Worker AQS总结

主要两个作用
1. 将锁的粒度细化到每个Worker。1. 直接使用CAS获取，避免阻塞。
### runWorker总结
- 如果线程池目前状态大于或等于STOP，会终止Worker中的线程。- 如果线程池此时是Running或者shutDown。会先去执行Worker的firstTask，如果firstTask执行结束或者为空，则会循环去执行工作队列中的任务，如果工作队列为空，会阻塞住。