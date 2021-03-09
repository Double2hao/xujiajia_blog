#Java Timer源码解析（定时器源码解析）
# Timer概述

Timer顾名思义就是定时器，用于处理一些需要延时处理的任务，延时时间可能是1s，也可能是5天。 一般使用方式如下：

```
        TimerTask task = new TimerTask() {
            @Override
            public void run() {
                Log.d("test", "timer task test");
            }
        };
        Timer timer = new Timer();
        timer.schedule(task, new Date(), 1000);

```

# 初始化

从上面代码看来，主要是Timer这个类，首先看一下Timer的初始化：

```
    public Timer() {
        this("Timer-" + serialNumber());
    }

```

```
    public Timer(String name) {
        thread.setName(name);
        thread.start();
    }

```

```
    private final TimerThread thread = new TimerThread(queue);

```

由上面代码，我们可以知道，Timer初始化其实就是start了TimerThread这个线程，于是我们看一下这个线程做的工作：

```
public void run() {
        try {
            mainLoop();
        } finally {
            // Someone killed this Thread, behave as if Timer cancelled
            synchronized(queue) {
                newTasksMayBeScheduled = false;
                queue.clear();  // Eliminate obsolete references
            }
        }
    }
    private void mainLoop() {
        while (true) {
            try {
                TimerTask task;
                boolean taskFired;
                synchronized(queue) {
                    // Wait for queue to become non-empty
                    while (queue.isEmpty() &amp;&amp; newTasksMayBeScheduled)
                        queue.wait();
                    if (queue.isEmpty())
                        break; // Queue is empty and will forever remain; die

                    // Queue nonempty; look at first evt and do the right thing
                    long currentTime, executionTime;
                    task = queue.getMin();
                    synchronized(task.lock) {
                        if (task.state == TimerTask.CANCELLED) {
                            queue.removeMin();
                            continue;  // No action required, poll queue again
                        }
                        currentTime = System.currentTimeMillis();
                        executionTime = task.nextExecutionTime;
                        if (taskFired = (executionTime&lt;=currentTime)) {
                            if (task.period == 0) { // Non-repeating, remove
                                queue.removeMin();
                                task.state = TimerTask.EXECUTED;
                            } else { // Repeating task, reschedule
                                queue.rescheduleMin(
                                  task.period&lt;0 ? currentTime   - task.period
                                                : executionTime + task.period);
                            }
                        }
                    }
                    if (!taskFired) // Task hasn't yet fired; wait
                        queue.wait(executionTime - currentTime);
                }
                if (taskFired)  // Task fired; run it, holding no locks
                    task.run();
            } catch(InterruptedException e) {
            }
        }
    }

```

代码不长，也比较容易看懂： 1、查看任务队列是否为空，如果为空就阻塞住。 2、获取队列中等待时间最少的一个，查看其状态是否已被关闭以及是否需要重复执行。 3、如果任务没有被取消，不需要移除队列。那么就判断其需要运行的时间是否小于等于当前时间，如果小于等于，就通过wait等待到该时间执行，如果大于，那么就直接run执行。

# 任务队列（小根堆）

定时器每次都会触发时间最小的那个任务，这种取极值的情况用堆非常的合适。 使用堆结构主要快在添加和删除，其性能都是log(n)。 也就是说在1000个任务的堆中添加和删除操作都能在10次对比以内完成。

源码如下：

```
class TaskQueue {

    private TimerTask[] queue = new TimerTask[128];
    private int size = 0;

    int size() {
        return size;
    }

    void add(TimerTask task) {
        // Grow backing store if necessary
        if (size + 1 == queue.length)
            queue = Arrays.copyOf(queue, 2*queue.length);

        queue[++size] = task;
        fixUp(size);
    }

    TimerTask getMin() {
        return queue[1];
    }

    TimerTask get(int i) {
        return queue[i];
    }

    void removeMin() {
        queue[1] = queue[size];
        queue[size--] = null;  // Drop extra reference to prevent memory leak
        fixDown(1);
    }

    void quickRemove(int i) {
        assert i &lt;= size;

        queue[i] = queue[size];
        queue[size--] = null;  // Drop extra ref to prevent memory leak
    }

    void rescheduleMin(long newTime) {
        queue[1].nextExecutionTime = newTime;
        fixDown(1);
    }

    boolean isEmpty() {
        return size==0;
    }

    void clear() {
        // Null out task references to prevent memory leak
        for (int i=1; i&lt;=size; i++)
            queue[i] = null;

        size = 0;
    }

    private void fixUp(int k) {
        while (k &gt; 1) {
            int j = k &gt;&gt; 1;
            if (queue[j].nextExecutionTime &lt;= queue[k].nextExecutionTime)
                break;
            TimerTask tmp = queue[j];  queue[j] = queue[k]; queue[k] = tmp;
            k = j;
        }
    }

    private void fixDown(int k) {
        int j;
        while ((j = k &lt;&lt; 1) &lt;= size &amp;&amp; j &gt; 0) {
            if (j &lt; size &amp;&amp;
                queue[j].nextExecutionTime &gt; queue[j+1].nextExecutionTime)
                j++; // j indexes smallest kid
            if (queue[k].nextExecutionTime &lt;= queue[j].nextExecutionTime)
                break;
            TimerTask tmp = queue[j];  queue[j] = queue[k]; queue[k] = tmp;
            k = j;
        }
    }
    void heapify() {
        for (int i = size/2; i &gt;= 1; i--)
            fixDown(i);
    }
}

```

Timer源码中的堆实现非常基础，主要是两个方法，fixUp()和fixDown()，即上升和下沉。 fixUp：为了保证堆结构，在addTask的时候会使用到上升。 fixDown：在取出最小值的时候，会把堆的最后一个节点放到堆顶，然后执行下沉操作，从而继续保证堆结构，将堆中剩余的值的最小值放到堆顶。
