#CountDownLatch 概述和源码分析
# 概述

源码中对这个类的描述如下：

>  
 A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes. 


意思大概是：

>  
 它是一个线程同步的助手，能够让一个或者多个线程在一组操作完成之前等待。 


# 简单场景（例子）

现在有10个人开会，在10人人全部到达会议室之前，“开会”这个线程就要一直等待。

## 代码：

```
import java.util.concurrent.CountDownLatch;

public class CountDownLatchTest {

    public static void main(String[] args) throws InterruptedException {

        final CountDownLatch countDownLatch = new CountDownLatch(10);

        for (int i = 0; i &lt; 10; i++) {
            final int Number = i + 1;

            new Thread(new Runnable() {
                public void run() {
                    try {
                        Thread.sleep((long) (Math.random() * 3000));
                        System.out.println("No." + Number + " arrived");
                    } catch (InterruptedException e) {
                    } finally {
                        countDownLatch.countDown();
                        System.out.println("CountDownLatch.count:" +countDownLatch.getCount());
                    }
                }
            }).start();
        }

        System.out.println("Wait");
        countDownLatch.await();
        System.out.println("Start");
    }
}
```

## 执行结果：

```
Wait
No.7 arrived
CountDownLatch.count:9
No.10 arrived
CountDownLatch.count:8
No.4 arrived
CountDownLatch.count:7
No.5 arrived
CountDownLatch.count:6
No.6 arrived
CountDownLatch.count:5
No.8 arrived
CountDownLatch.count:4
No.1 arrived
CountDownLatch.count:3
No.9 arrived
CountDownLatch.count:2
No.2 arrived
CountDownLatch.count:1
No.3 arrived
CountDownLatch.count:0
Start

```

# 源码分析

单单CountDownLatch这个类的源码其实非常少，源码如下：

```
public class CountDownLatch {<!-- -->
    private static final class Sync extends AbstractQueuedSynchronizer {<!-- -->
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c - 1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }

    private final Sync sync;

    public CountDownLatch(int count) {
        if (count &lt; 0) throw new IllegalArgumentException("count &lt; 0");
        this.sync = new Sync(count);
    }
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }
    public void countDown() {
        sync.releaseShared(1);
    }
    public long getCount() {
        return sync.getCount();
    }
    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```

首先我们可以看到AbstractQueuedSynchronizer这个比较显眼的类，它的源码有2000多行，比较多，要全部看完比较耗时间。  因此，我们主要还是从我们经常使用的几个方法看起：  1、构造函数  2、await()方法  3、countDown()方法

## 构造函数

由源码最终可知，构造函数最终调用了AbstractQueuedSynchronizer类的setState(count)方法：

```
    protected final void setState(int newState) {
        state = newState;
    }
```

```
    /**
     * The synchronization state.
     */
    private volatile int state;
```

从AbstractQueuedSynchronizer源码中我们可以看到state就是一个同步状态。

## await()

由CountDownLatch源码可知，await()调用了AbstractQueuedSynchronizer的tryAcquireSharedNanos方法：

```
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) &lt; 0)
            doAcquireSharedInterruptibly(arg);
    }
```

这里可以看到如果中断就会抛异常，最终如果执行成功会执行到doAcquireSharedInterruptibly这个方法：

```
/**
     * Acquires in shared interruptible mode.
     * @param arg the acquire argument
     */
    private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r &gt;= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &amp;&amp;
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } catch (Throwable t) {
            cancelAcquire(node);
            throw t;
        }
    }
```

Node在这里我们看做是一个等待队列的链表就可以了，在等待结束后会从等待队列中依次取对象出来执行。比如上面的例子中，等待事件就是“开会”。  这边的parkAndCheckInterrupt()方法就是阻塞线程的方法,源码如下：

```
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

## countDown()

由CountDownLatch源码可知，await()调用了AbstractQueuedSynchronizer的releaseShared方法：

```
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```

这边主要是tryReleaseShared和doReleaseShared两个方法：

```
protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }

```

tryReleaseShared方法很简单，使用CAS更改state的值。

```
private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null &amp;&amp; h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!h.compareAndSetWaitStatus(Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &amp;&amp;
                         !h.compareAndSetWaitStatus(0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

doReleaseShared的作用主要是来唤醒阻塞线程。

doReleaseShared里是一个死循环，只有Node链表中只有一个head时才会跳出循环。  unparkSuccessor就是唤醒被阻塞的线程的方法。  在head存在并且状态为 waitStatus时，会通过CAS将状态变为0，并且唤醒阻塞的线程。