#ReentrantLock和AQS 源码解析
# 概要

ReentrantLock是一种经常会被用来与Synchronized比较的一种同步机制，其在Java中的应用也十分广泛，比如最常见的BlockingQueue就是使用了ReentrantLock来实现的同步机制。

>  
 个人认为ReentrantLock和Synchronized的区别主要有以下三点： 
 - ReentrantLock等待可中中断- ReentrantLock可以实现公平锁- ReentrantLock可以通过Condition绑定多个条件 


# 经典Demo

本文打算以常用的几个方法为切入点来分析源码，首先看下ArrayBlockingQueue源码中ReentrantLock的经典使用方式： **put方法：**
1. 上锁1. 如果发现queue满了就阻塞至队列不满为止。然后将对象加入队列。1. 最终解锁
**take方法：**
1. 上锁1. 如果发现queue为空就阻塞至队列不为空为止。然后将队列中值导出。1. 最终解锁
```
private final Condition notFull;
private final Condition notEmpty;
public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity &lt;= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }
    
public void put(E e) throws InterruptedException {
        Objects.requireNonNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
    
public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    
private void enqueue(E e) {
        final Object[] items = this.items;
        items[putIndex] = e;
        if (++putIndex == items.length) putIndex = 0;
        count++;
        notEmpty.signal();
    }
private E dequeue() {
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E e = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length) takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();
        return e;
    }
private void insert(E x) {
        items[putIndex] = x;
        putIndex = inc(putIndex);
        ++count;
        notEmpty.signal();
 }

```

### 主要方法及作用

**ReentrantLock.lock():**(和lockInterruptibly实现大致相同) 用来获取锁。

**ReentrantLock.unlock():** 解锁。

**Condition.await():** 释放锁，并且阻塞。只有当该Condition被signal的时候才会停止阻塞。

**Condition.signal():** 唤醒执行了Condition.await()的线程。

# 阻塞唤醒数据结构

在开始分析各个主要方法的源码之前，首先要讲一下重要的实现阻塞唤醒的数据结构：

**AbstractQueuedSynchronizer.Node** Node是一个等待队列，AQS就是通过这个队列来处理“锁争抢”的问题的，在有多个线程产生“锁争抢”的情况的时候，会根据对象在Node队列中的位置去获取锁。Node中存储了每一个等待对象当前的状态。

在ReentrantLock中有两个地方用到了Node：
1. 每个ReentrantLock对象对应一个Node队列。如果lock获取锁失败，在阻塞线程的同时，会生成一个Node对象到等待队列中。1. 每个Condition对象对应一个Node队列。在执行await方法的时候，会生成一个Node对象加入到等待队列中。
```
	/**
     * Wait queue node class.
     *
     */
		static final class Node {
        static final Node SHARED = new Node();
        
        /** Marker to indicate a node is waiting in exclusive mode */
        static final Node EXCLUSIVE = null;

        /** waitStatus value to indicate thread has cancelled. */
        static final int CANCELLED =  1;
        
        /** waitStatus value to indicate successor's thread needs unparking. */
        static final int SIGNAL    = -1;
        
        /** waitStatus value to indicate thread is waiting on condition. */
        static final int CONDITION = -2;
        
        /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate.
         */
        static final int PROPAGATE = -3;
        
        volatile int waitStatus;
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
        Node nextWaiter;
        
        //其他部分省略
}

```

# ReentrantLock.lock()

主要过程如下：
1. 尝试用CAS去获取锁，如果获取到锁，就会把当前获取到锁的线程设置为自己，并且返回true。(实现了可重入锁，如果同一个线程反复获取同一个锁不会阻塞)1. 如果没有获取到锁，那么会进入到acquireQueued()的自旋中，只有获取到锁或者中断异常了才会跳出。
```
    public void lock() {
        sync.acquire(1);
    }

```

```
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &amp;&amp;
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

```

```
		protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
        
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc &lt; 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

```

### acquireQueued()自旋

自旋中有两个if：
1. 如果前驱节点为head，那么就尝试获取锁。 当前驱节点为head时，说明前面没有其他等待获取锁的对象，后面就轮到当前节点来获取锁了。1. 如果需要阻塞，那么就阻塞。 当还没有轮到自己来获取锁的时候，就通过LockSupport.park(this)这个方法来阻塞。但是，由于ReentrantLock本身是支持停止阻塞的，所以有可能在被唤醒的时候线程已经由于超时等原因被中断了。
```
    final boolean acquireQueued(final Node node, int arg) {
        boolean interrupted = false;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head &amp;&amp; tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node))
                    interrupted |= parkAndCheckInterrupt();
            }
        } catch (Throwable t) {
            cancelAcquire(node);
            if (interrupted)
                selfInterrupt();
            throw t;
        }
    }
    
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }

```

# ReentrantLock.unlock()

主要过程如下：
1. 如果锁当前被占有，直接释放，返回true。如果锁已经释放，就返回false。1. 当锁释放之后，会尝试去唤醒在Node队列中排在此Node后面的一个线程。
```
    public void unlock() {
        sync.release(1);
    }

```

```
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null &amp;&amp; h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    
	protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

```

```
    private void unparkSuccessor(Node node) {
        int ws = node.waitStatus;
        if (ws &lt; 0)
            node.compareAndSetWaitStatus(ws, 0);

        Node s = node.next;
        if (s == null || s.waitStatus &gt; 0) {
            s = null;
            for (Node p = tail; p != node &amp;&amp; p != null; p = p.prev)
                if (p.waitStatus &lt;= 0)
                    s = p;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }

```

# Condition.await()

主要过程如下：
1. 添加新的Node到Condition的等待队列中1. 释放掉AQS占有的锁。1. 使用LockSupport.park(this)阻塞住当前线程，只有当Condition.singal()或者线程被中断的时候才会执行后续逻辑。1. 如果线程已经被中断，会执行一些清理操作。
```
        public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            Node node = addConditionWaiter();
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            if (acquireQueued(node, savedState) &amp;&amp; interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }

```

```
        private Node addConditionWaiter() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node t = lastWaiter;
            // If lastWaiter is cancelled, clean out.
            if (t != null &amp;&amp; t.waitStatus != Node.CONDITION) {
                unlinkCancelledWaiters();
                t = lastWaiter;
            }

            Node node = new Node(Node.CONDITION);

            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node;
            return node;
        }

```

```
    final int fullyRelease(Node node) {
        try {
            int savedState = getState();
            if (release(savedState))
                return savedState;
            throw new IllegalMonitorStateException();
        } catch (Throwable t) {
            node.waitStatus = Node.CANCELLED;
            throw t;
        }
    }
    
	public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null &amp;&amp; h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }


```

# Condition.signal()

主要逻辑如下：
1. 首先判断Condition对应的等待队列的第一个Node是否为空，如果不为空就执行后续逻辑。1. 使用LockSupport.unpark(node.thread)唤醒Condition等待队列中的第一个Node对应的线程，在唤醒之后在等待队列中逻辑删除第一个Node。
```
        public final void signal() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
                doSignal(first);
        }

```

```
        private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &amp;&amp;
                     (first = firstWaiter) != null);
        }

```

```
    final boolean transferForSignal(Node node) {
        if (!node.compareAndSetWaitStatus(Node.CONDITION, 0))
            return false;

		enq(node);
        int ws = p.waitStatus;
        if (ws &gt; 0 || !p.compareAndSetWaitStatus(ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }

```

# ReentrantLock公平锁

默认的ReentrantLock和Synchronized都是非公平锁的。 那么为什么要在ReentrantLock中引入公平锁？

```
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    
	static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }

```

我们可以直接来看下两者源码的区别。

```
		protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
        
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc &lt; 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

```

```
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        @ReservedStackAccess
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &amp;&amp;
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc &lt; 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }

```

可以看到，其实公平锁与非公平锁相比，就是在CAS之前多了个hasQueuedPredecessors()的判断。

```
/**
     * Queries whether any threads have been waiting to acquire longer
     * than the current thread.
     * /
    public final boolean hasQueuedPredecessors() {
    
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &amp;&amp;
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }

```

这个方法的作用在源码注释中已经写得很清楚：判断是否有其他线程比当前线程等待得更久。

在公平锁中，如果有其他线程比当前线程等待的更久，那么当前线程就会直接进入等待队列，而不会使用 CAS来获取锁。

>  
 为什么默认是非公平锁，而不是公平锁？ 答：因为非公平锁性能更好。 公平锁一旦发现有其他比自己等待的更久的线程，会直接阻塞当前线程，进入等待队列。而如果是非公平锁，在这种情况下，完全可能能通过CAS直接获取到锁。 线程的阻塞和唤醒本身会耗费一定资源，尤其在高并发的情况下，非公平锁与公平锁相比能够节省很多线程调度的资源。 
