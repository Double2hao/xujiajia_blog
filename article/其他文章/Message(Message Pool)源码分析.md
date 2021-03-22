#Message(Message Pool)源码分析
# 前提概要

笔者有一段时间写Message一直这么写：

```
Message msg=new Message();
msg.what=0;
handler.sendMessage(msg);

```

但是事实上，为了提高效率，我们应该复用Message：

```
Message msg=handler.obtainMessage();
msg.what=0;
handler.sendMessage(msg);

```

那么在这个obtainMessage()中究竟是如何实现Message的复用的呢？ 存储结构是怎样，线程并发如何处理等。

# 源码解析

## 获取逻辑

首先由obtainMessage()为入口，然后可以即进入到Message.obtain()方法：

```
    public final Message obtainMessage()
    {
        return Message.obtain(this);
    }

```

```
    public static Message obtain(Handler h) {
        Message m = obtain();
        m.target = h;

        return m;
    }

```

```
    public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }

```

```
    private static final Object sPoolSync = new Object();
    private static Message sPool;

```

从obtain()源码中我们可以看到，所谓的MessagePool，不过就是一个链表。

对线程并发的处理也便是用synchronized 对sPoolSync 这个final对象加锁。 而此处为什么不用对象锁或者类锁呢？ 由于是静态方法，对象可能不存在，所以不能用对象锁。 用类锁虽然也可以实现，但是MessagePool的逻辑和Message类本身关系不大，用了一方面不是很恰当，另一方面也限制了Message类的拓展。

## 回收逻辑

获取的逻辑基本就这样了，再看下回收的逻辑，这里我们直接看到MessageQueue.removeMessages()和其中调用到的Message.recycleUnchecked():

```
    void removeMessages(Handler h, int what, Object object) {
        if (h == null) {
            return;
        }

        synchronized (this) {
            Message p = mMessages;

            // Remove all messages at front.
            while (p != null &amp;&amp; p.target == h &amp;&amp; p.what == what
                   &amp;&amp; (object == null || p.obj == object)) {
                Message n = p.next;
                mMessages = n;
                p.recycleUnchecked();
                p = n;
            }

            // Remove all messages after front.
            while (p != null) {
                Message n = p.next;
                if (n != null) {
                    if (n.target == h &amp;&amp; n.what == what
                        &amp;&amp; (object == null || n.obj == object)) {
                        Message nn = n.next;
                        n.recycleUnchecked();
                        p.next = nn;
                        continue;
                    }
                }
                p = n;
            }
        }
    }

```

```
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            if (sPoolSize &lt; MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }

```

可以看到，recycleUnchecked()就是Message中另一处调用到synchronize(sPoolSync)的地方。对所谓“MessagePool”的处理也就这两处。recycleUnchecked()中的逻辑便是把Message中的内容清空。 在MessageQueue中removeMessage的逻辑非常简单，首先把要删除的Message清空，然后把链表中后面的节点放到前面来。（并没有删除对象，只是清空了内容放到链表后面而已）