#android LruCache源码解析
# LruCache

LRU为最近最少使用算法，LruCache顾名思义即为最近最少使用算法下的缓存机制。 LRU的目的是为了加快最近经常使用的数据从内存中取出的速度。

在android中LruCache在图片缓存中频繁使用到，了解它绝对是必要的。

# Lru算法实现

首先当然是看 LruCache这个类的源码，我们很容易发现LruCache类中仅仅是get，put等供开发者使用的方法，并未涉及链表等结构。（由于代码较长，笔者此处就不附上LruCache类的代码了） 但是其中却又一个我们不常见的类，即LinkedHashMap，经过查看，得出结论：

>  
 LruCache的底层是通过LinkedHashMap实现数据缓存的。 


LinkedHashMap是一个双向链表,继承了HashMap。

>  
 对HashMap不了解的可以看这篇文章：  


根据LRU算法的思路可知，LRU算法定然是会在取出数据时对链表进行操作，从而加快下一次取出“经常使用的数据”的速度。 于是，我们定然先查看get方法。

```
@Override public V get(Object key) {
        /*
         * This method is overridden to eliminate the need for a polymorphic
         * invocation in superclass at the expense of code duplication.
         */
        if (key == null) {
            HashMapEntry&lt;K, V&gt; e = entryForNullKey;
            if (e == null)
                return null;
            if (accessOrder)
                makeTail((LinkedEntry&lt;K, V&gt;) e);
            return e.value;
        }

        int hash = Collections.secondaryHash(key);
        HashMapEntry&lt;K, V&gt;[] tab = table;
        for (HashMapEntry&lt;K, V&gt; e = tab[hash &amp; (tab.length - 1)];
                e != null; e = e.next) {
            K eKey = e.key;
            if (eKey == key || (e.hash == hash &amp;&amp; key.equals(eKey))) {
                if (accessOrder)
                    makeTail((LinkedEntry&lt;K, V&gt;) e);
                return e.value;
            }
        }
        return null;
    }

```

除去key等于null等意外的情况不看，我们可以发现，最关键的其实是这几行代码：

```
for (HashMapEntry&lt;K, V&gt; e = tab[hash &amp; (tab.length - 1)];
                e != null; e = e.next) {
            K eKey = e.key;
            if (eKey == key || (e.hash == hash &amp;&amp; key.equals(eKey))) {
                if (accessOrder)
                    makeTail((LinkedEntry&lt;K, V&gt;) e);
                return e.value;
            }
        }

```

e即为我们要取出的那个结点，而makeTail在取出这个结点之前对其进行了操作，那么makeTail定然便是实现LRU的方法。 接下来我们跳转到makeTail这个方法中来。

```
 private void makeTail(LinkedEntry&lt;K, V&gt; e) {
        // Unlink e
        e.prv.nxt = e.nxt;
        e.nxt.prv = e.prv;

        // Relink e as tail
        LinkedEntry&lt;K, V&gt; header = this.header;
        LinkedEntry&lt;K, V&gt; oldTail = header.prv;
        e.nxt = header;
        e.prv = oldTail;
        oldTail.nxt = header.prv = e;
        modCount++;
    }

```

根据代码我们可知：把e这个结点从原链表中取出，放到链表的头结点。让它成为原链表的前一个结点，成为原链表的尾结点的最后一个结点。（原链表为双向链表） **所以链表中经常使用的结点都会被排在前面，而不经常使用的结点都会被排在后面。** 这时候我们再来看get中的关键代码：

```
for (HashMapEntry&lt;K, V&gt; e = tab[hash &amp; (tab.length - 1)];
                e != null; e = e.next) {
            K eKey = e.key;
            if (eKey == key || (e.hash == hash &amp;&amp; key.equals(eKey))) {
                if (accessOrder)
                    makeTail((LinkedEntry&lt;K, V&gt;) e);
                return e.value;
            }
        }

```

我们可以发现，get中查找结点的方式是**从前向后找**的。 因此**经常使用的结点查找时间较短，而不经常使用的结点查找时间较长**。完全符合LRU算法的最终目的。

# 如何回收不经常使用的内存

这时候我们似乎对LruCache是如何运作的已经有了一个了解了，但是心中似乎还有一个梗：

>  
 内存是有限的，当内存满了的时候，LruCache是如何回收不经常使用的内存的？ 


其实思路很清晰，我们分配给LruCache的内存大小是从哪里传进去的，从那里定能寻找到踪迹。于是便是来看LruCache的构造方法了。

```
public LruCache(int maxSize) {
        if (maxSize &lt;= 0) {
            throw new IllegalArgumentException("maxSize &lt;= 0");
        }
        this.maxSize = maxSize;
        this.map = new LinkedHashMap&lt;K, V&gt;(0, 0.75f, true);
    }

```

我们发现传进来的内存LruCache中用maxSize 这个属性进行存储。那么我们只要找到maxSize 具体使用的地方便可以了。 于是我们可以发现在resize和put方法中都有使用到，并且都是把maxSize 传入了trimToSize()这个方法中使用的。

```
public void resize(int maxSize) {
        if (maxSize &lt;= 0) {
            throw new IllegalArgumentException("maxSize &lt;= 0");
        }

        synchronized (this) {
            this.maxSize = maxSize;
        }
        trimToSize(maxSize);
    }

```

```
public final V put(K key, V value) {
        if (key == null || value == null) {
            throw new NullPointerException("key == null || value == null");
        }

        V previous;
        synchronized (this) {
            putCount++;
            size += safeSizeOf(key, value);
            previous = map.put(key, value);
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }

        if (previous != null) {
            entryRemoved(false, key, previous, value);
        }

        trimToSize(maxSize);
        return previous;
    }

```

于是我们可知，trimToSize（）这个方法便是内存回收的关键。我们便来看一看trimToSize（）这个方法。

```
public void trimToSize(int maxSize) {
        while (true) {
            K key;
            V value;
            synchronized (this) {
                if (size &lt; 0 || (map.isEmpty() &amp;&amp; size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }

                if (size &lt;= maxSize) {
                    break;
                }

                Map.Entry&lt;K, V&gt; toEvict = map.eldest();
                if (toEvict == null) {
                    break;
                }

                key = toEvict.getKey();
                value = toEvict.getValue();
                map.remove(key);
                size -= safeSizeOf(key, value);
                evictionCount++;
            }

            entryRemoved(true, key, value, null);
        }
    }

```

其中map是一个LinkedHashMap对象，为了方便后面的分析，我们放上它的eldest方法代码，其实就是取出最久未使用的那个结点。

```
 public Entry&lt;K, V&gt; eldest() {
        LinkedEntry&lt;K, V&gt; eldest = header.nxt;
        return eldest != header ? eldest : null;
    }

```

trimToSize（）的内存回收逻辑主要是以下几行代码：

```
				Map.Entry&lt;K, V&gt; toEvict = map.eldest();
                if (toEvict == null) {
                    break;
                }

                key = toEvict.getKey();
                value = toEvict.getValue();
                map.remove(key);

```

即查找到最久未使用的结点并remove。 那么这个逻辑究竟在什么情况下会执行呢？这段代码之前的两个if给了我们很好的解释： 1、内存中有数据 2、内存中的数据超过了maxSize即提供的最大内存量。

至于trimToSize（）的while和synchronized 其实都比较好理解 while是为了让内存一直低于maxSize即最大内存。 synchronized 保证了线程安全。
