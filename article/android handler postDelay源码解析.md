#android handler postDelay源码解析
# 概述

postDelay是在android中经常用来处理时延任务的操作。 近期突然比较好奇postDelay实现时延的原理，于是学习后作此文。

>  
 如果对handler原理还完全不了解的读者可以看下笔者的此篇文章： 


# postDelay的调用过程

```
	public final boolean postDelayed(@NonNull Runnable r, long delayMillis) {<!-- -->
        return sendMessageDelayed(getPostMessage(r), delayMillis);
    }

```

```
    public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {<!-- -->
        if (delayMillis &lt; 0) {<!-- -->
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

```

```
    public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {<!-- -->
        MessageQueue queue = mQueue;
        if (queue == null) {<!-- -->
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }

```

```
    private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
            long uptimeMillis) {<!-- -->
        msg.target = this;
        msg.workSourceUid = ThreadLocalWorkSource.getUid();

        if (mAsynchronous) {<!-- -->
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

```

# MessageQueue.enqueueMessage()

最终可以看到是进入了MessageQueue的enqueueMessage方法中。 enqueueMessage方法中的逻辑如下：
1. 针对各种异常情况做处理1. message根据触发时间的从小到大排列。1. 根据mBlocked来判断是否需要触发nativeWake()来唤醒被阻塞的消息队列。
>  
 这里通过注释也可以看到。 一般情况下，只有立即触发的消息才会调用到nativeWake()，而时延的消息则不会触发。 


```
    boolean enqueueMessage(Message msg, long when) {<!-- -->
        if (msg.target == null) {<!-- -->
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {<!-- -->
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {<!-- -->
            if (mQuitting) {<!-- -->
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when &lt; p.when) {<!-- -->
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {<!-- -->
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked &amp;&amp; p.target == null &amp;&amp; msg.isAsynchronous();
                Message prev;
                for (;;) {<!-- -->
                    prev = p;
                    p = p.next;
                    if (p == null || when &lt; p.when) {<!-- -->
                        break;
                    }
                    if (needWake &amp;&amp; p.isAsynchronous()) {<!-- -->
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {<!-- -->
                nativeWake(mPtr);
            }
        }
        return true;
    }

```

# MessageQueue.next()

在了解了looper.loop()的源码之后，我们知道，如果没有消息，那么loop()会阻塞在MessageQueue的next()方法中，接下来我们就需要根据这个关键的方法来进一步解析。

>  
 如果不了解looper源码的读者，可以看下笔者开头的文章。 


```
    Message next() {<!-- -->
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {<!-- -->
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {<!-- -->
            if (nextPollTimeoutMillis != 0) {<!-- -->
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {<!-- -->
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null &amp;&amp; msg.target == null) {<!-- -->
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {<!-- -->
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null &amp;&amp; !msg.isAsynchronous());
                }
                if (msg != null) {<!-- -->
                    if (now &lt; msg.when) {<!-- -->
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {<!-- -->
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {<!-- -->
                            prevMsg.next = msg.next;
                        } else {<!-- -->
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {<!-- -->
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {<!-- -->
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount &lt; 0
                        &amp;&amp; (mMessages == null || now &lt; mMessages.when)) {<!-- -->
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount &lt;= 0) {<!-- -->
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {<!-- -->
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i &lt; pendingIdleHandlerCount; i++) {<!-- -->
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {<!-- -->
                    keep = idler.queueIdle();
                } catch (Throwable t) {<!-- -->
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {<!-- -->
                    synchronized (this) {<!-- -->
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }

```

### MessageQueue.IdleHandler

这里发现IdleHandler这个类完全没见过，于是看下它的源码。

```
    /**
     * Callback interface for discovering when a thread is going to block
     * waiting for more messages.
     */
    public static interface IdleHandler {<!-- -->
        /**
         * Called when the message queue has run out of messages and will now
         * wait for more.  Return true to keep your idle handler active, false
         * to have it removed.  This may be called if there are still messages
         * pending in the queue, but they are all scheduled to be dispatched
         * after the current time.
         */
        boolean queueIdle();
    }

```

从注释和next()的逻辑中可以看到，在message被阻塞的时候IdleHandler.queueIdle()会被调用到。并且会根据IdleHandler.queueIdle()的返回值来确认是否要从mIdleHandlers的list中移除。

### MessageQueue.next()总结

这样整个next()的逻辑就较为清晰了，主要逻辑如下。
1. 进来比较关键的点是一个for循环，进入后就会执行执行nativePollOnce方法。如果上一轮的循环中，message是时延执行的，那么这轮循环进入后，会阻阻塞nextPollTimeoutMillis的时间再执行。1. 如果是时延的message，会用nextPollTimeoutMillis记录下下次执行的时间。如果此时已经到了执行message的时间，那么会直接将此message返回。1. 如果此时是时延的message，那么会调用IdleHandler.queueIdle()方法。