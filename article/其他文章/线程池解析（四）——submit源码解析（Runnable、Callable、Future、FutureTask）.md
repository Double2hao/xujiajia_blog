#线程池解析（四）——submit源码解析（Runnable、Callable、Future、FutureTask）
>  
 <h2>相关文章</h2> 
     


# 概述

线程池的submit也是常用的方法之一，它有很多个重载，涉及到了很多个类。 本文将探索submit方法以及相关的各个类。

# submit源码

我们看下ThreadPoolExecutor中sumit相关的源码。

```
    protected &lt;T&gt; RunnableFuture&lt;T&gt; newTaskFor(Runnable runnable, T value) {<!-- -->
        return new FutureTask&lt;T&gt;(runnable, value);
    }

    protected &lt;T&gt; RunnableFuture&lt;T&gt; newTaskFor(Callable&lt;T&gt; callable) {<!-- -->
        return new FutureTask&lt;T&gt;(callable);
    }

    public Future&lt;?&gt; submit(Runnable task) {<!-- -->
        if (task == null) throw new NullPointerException();
        RunnableFuture&lt;Void&gt; ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }

    public &lt;T&gt; Future&lt;T&gt; submit(Runnable task, T result) {<!-- -->
        if (task == null) throw new NullPointerException();
        RunnableFuture&lt;T&gt; ftask = newTaskFor(task, result);
        execute(ftask);
        return ftask;
    }

    public &lt;T&gt; Future&lt;T&gt; submit(Callable&lt;T&gt; task) {<!-- -->
        if (task == null) throw new NullPointerException();
        RunnableFuture&lt;T&gt; ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }

```

```
    protected &lt;T&gt; RunnableFuture&lt;T&gt; newTaskFor(Callable&lt;T&gt; callable) {<!-- -->
        return new FutureTask&lt;T&gt;(callable);
    }
        protected &lt;T&gt; RunnableFuture&lt;T&gt; newTaskFor(Runnable runnable, T value) {<!-- -->
        return new FutureTask&lt;T&gt;(runnable, value);
    }

```

只看submit直接相关的一些代码发现其实已经包含了本文中所有的需要讲解的类：Runnable、Callable、Future、RunnableFuture、FutureTask 但是只看这些源码只是真正使用的地方，需要具体了解还是要看下具体的各个类的源码。

# 几个容易混淆的类

### Runnable

```
public interface Runnable {<!-- -->
    public abstract void run();
}

```

Runnable很常见，只有一个run方法。 使用Runnable，用户可以把需要运行的逻辑封装到这个类中，稍后执行。

### Callable

```
public interface Callable&lt;V&gt; {<!-- -->
    V call() throws Exception;
}

```

Callable与Runnable相比就是多了返回值和 抛出异常的逻辑。 用户也是可以把需要运行的逻辑封装到这个类中，在自己需要的时候执行。

### Future

```
public interface Future&lt;V&gt; {<!-- -->

    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

```

通过Future，用户可以阻塞等待自己想要的结果，并且设置超时时间。 也可以查看该任务是否执行完毕，是否被取消，以及取消该逻辑。

### RunnableFuture

```
public interface RunnableFuture&lt;V&gt; extends Runnable, Future&lt;V&gt; {<!-- -->
    void run();
}

```

RunnableFuture就是继承了Runnable与Future，拥有两者的特性。

# FutureTask

```
public class FutureTask&lt;V&gt; implements RunnableFuture&lt;V&gt;

```

FutureTask就是比较大的模块了，可以很直观的看到FutureTask是继承的RunnableFuture。 因此我们可以将逻辑分为两块，一块是Runnable接口的实现，另一块是Future接口的实现。 但是在分析之前，我准备先讲下FutureTask的各种状态，这是FutureTask非常基础的一部分，其源码的实现都依赖了这些状态。

## FutureTask的状态

```
    private volatile int state;
    private static final int NEW          = 0;
    private static final int COMPLETING   = 1;
    private static final int NORMAL       = 2;
    private static final int EXCEPTIONAL  = 3;
    private static final int CANCELLED    = 4;
    private static final int INTERRUPTING = 5;
    private static final int INTERRUPTED  = 6;

```

相互间的转换可能如下：
- 正常执行结束，并且返回结果。 NEW -&gt; COMPLETING -&gt; NORMAL- 运行的时候产生了异常。 NEW -&gt; COMPLETING -&gt; EXCEPTIONAL- 定义了task，在运行之前取消了。 NEW -&gt; CANCELLED- cancel的时候，task已经在运行了。 NEW -&gt; INTERRUPTING -&gt; INTERRUPTED
## FutureTask中Runnable接口的实现

```
    public void run() {<!-- -->
        if (state != NEW ||
            !U.compareAndSwapObject(this, RUNNER, null, Thread.currentThread()))
            return;
        try {<!-- -->
            Callable&lt;V&gt; c = callable;
            if (c != null &amp;&amp; state == NEW) {<!-- -->
                V result;
                boolean ran;
                try {<!-- -->
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {<!-- -->
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {<!-- -->
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s &gt;= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
    

```

```
    protected void set(V v) {<!-- -->
        if (U.compareAndSwapInt(this, STATE, NEW, COMPLETING)) {<!-- -->
            outcome = v;
            U.putOrderedInt(this, STATE, NORMAL); // final state
            finishCompletion();
        }
    }
    
    private void finishCompletion() {<!-- -->
        // assert state &gt; COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {<!-- -->
            if (U.compareAndSwapObject(this, WAITERS, q, null)) {<!-- -->
                for (;;) {<!-- -->
                    Thread t = q.thread;
                    if (t != null) {<!-- -->
                        q.thread = null;
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next;
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
                    q = next;
                }
                break;
            }
        }

        done();

        callable = null;        // to reduce footprint
    }

```

逻辑非常清晰，主要是以下几步：
1. 通过state来判断FutureTask的状态，只有是NEW的情况下才会继续执行。1. 执行构造中传入的callable，如果有异常就记录下，如果运行成功，也会将result记录下来。1. 设置result结束后，会通过LockSupport.unpark()来结束目前阻塞的线程。（此处可能会有多个线程阻塞）
## FutureTask中Future接口的实现

Future接口中，我们主要看下get和cancel这两个主要的接口。

### get源码

```
    public V get() throws InterruptedException, ExecutionException {<!-- -->
        int s = state;
        if (s &lt;= COMPLETING)
            s = awaitDone(false, 0L);
        return report(s);
    }

    public V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {<!-- -->
        if (unit == null)
            throw new NullPointerException();
        int s = state;
        if (s &lt;= COMPLETING &amp;&amp;
            (s = awaitDone(true, unit.toNanos(timeout))) &lt;= COMPLETING)
            throw new TimeoutException();
        return report(s);
    }

```

```
    private V report(int s) throws ExecutionException {<!-- -->
        Object x = outcome;
        if (s == NORMAL)
            return (V)x;
        if (s &gt;= CANCELLED)
            throw new CancellationException();
        throw new ExecutionException((Throwable)x);
    }

```

get的主要逻辑：
1. 先通过state判断当前认识是否结束，如果没有结束就执行awaitDone方法。1. 执行完毕后通过reorpt返回result值。 report中只有当前状态是NORMAL的情况才会正常返回result。或者如果是CANCELED状态会返回取消异常。
### awaitDone源码

```
    private int awaitDone(boolean timed, long nanos)
        throws InterruptedException {<!-- -->
        long startTime = 0L;    // Special value 0L means not yet parked
        WaitNode q = null;
        boolean queued = false;
        for (;;) {<!-- -->
            int s = state;
            if (s &gt; COMPLETING) {<!-- -->
                if (q != null)
                    q.thread = null;
                return s;
            }
            else if (s == COMPLETING)
                // We may have already promised (via isDone) that we are done
                // so never return empty-handed or throw InterruptedException
                Thread.yield();
            else if (Thread.interrupted()) {<!-- -->
                removeWaiter(q);
                throw new InterruptedException();
            }
            else if (q == null) {<!-- -->
                if (timed &amp;&amp; nanos &lt;= 0L)
                    return s;
                q = new WaitNode();
            }
            else if (!queued)
                queued = U.compareAndSwapObject(this, WAITERS,
                                                q.next = waiters, q);
            else if (timed) {<!-- -->
                final long parkNanos;
                if (startTime == 0L) {<!-- --> // first time
                    startTime = System.nanoTime();
                    if (startTime == 0L)
                        startTime = 1L;
                    parkNanos = nanos;
                } else {<!-- -->
                    long elapsed = System.nanoTime() - startTime;
                    if (elapsed &gt;= nanos) {<!-- -->
                        removeWaiter(q);
                        return state;
                    }
                    parkNanos = nanos - elapsed;
                }
                // nanoTime may be slow; recheck before parking
                if (state &lt; COMPLETING)
                    LockSupport.parkNanos(this, parkNanos);
            }
            else
                LockSupport.park(this);
        }
    }

```

awaitDone里通过一个for(;;)来处理所有的逻辑，因此代码的先后关系不代表执行的先后顺序：
- 如果已经执行结束，直接返回当前状态。- 如果此时状态是COMPLETING，会直接让出当前线程，稍后执行。（原因是COMPLETING是一个中间态，只有正在设置result或者正在设置异常的时候回执行）- 如果此时当前线程已经结束，直接抛异常，并且清理FutureTask的WaitNode。- 当不存在等待线程并且当前方法存在超时逻辑时，如果已经超时，那么直接返回当前state。 如果当前线程需要等待，但是FutureTask中还没有waitNode，那么会创建一个。- 如果当前线程将进入FutureTask的等待队列，那么需要通过CAS在waiters这个属性中记录一下。- 如果此时已经在FutureTask中通过FutureTask记录过自己的线程等待状态，并且此处自己需要等到的话，那么会通过LockSupport.parkNanos（）来阻塞当前线程，进行等待。- 如果没有超时逻辑，那么就会直接通过LockSupport.park()来阻塞当前线程进行等待。
# FutureTask构造中对runnable的处理

```
    public FutureTask(Callable&lt;V&gt; callable) {<!-- -->
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       
    }
    
    public FutureTask(Runnable runnable, V result) {<!-- -->
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;      
    }

```

这里有个很有意思的点：不管构造的时候传入的是runnable还是callable，在futureTask中都是存储为callable的。

我们可以看下具体实现的逻辑。

```
    public static &lt;T&gt; Callable&lt;T&gt; callable(Runnable task, T result) {<!-- -->
        if (task == null)
            throw new NullPointerException();
        return new RunnableAdapter&lt;T&gt;(task, result);
    }

```

```
    private static final class RunnableAdapter&lt;T&gt; implements Callable&lt;T&gt; {<!-- -->
        private final Runnable task;
        private final T result;
        RunnableAdapter(Runnable task, T result) {<!-- -->
            this.task = task;
            this.result = result;
        }
        public T call() {<!-- -->
            task.run();
            return result;
        }
    }

```

可以看到这里是通过RunnableAdapter将Runnable和result转化成了一个假的Callable，这个callable的call()的结果永远是result。