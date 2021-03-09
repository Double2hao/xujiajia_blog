#ThreadLocal与ThreadLocalMap源码解析
>  
 文章个人见解较多，如有问题欢迎指出。 


# 使用场景

一般来说，某些数据是**以线程为作用域**并且不同线程具有不同的数据的数据副本的时候，就可以考虑采用ThreadLocal。 比如Looper 内部就是通过ThreadLocal来保证不同线程中有不同的唯一副本的，从而实现了我们现在经常使用的android消息机制。

>  
 对于android消息机制不了解，并且好奇的同学可以先看一下笔者的博文：  


# 文章摘要

1、ThreadLocal内部就是通过操作ThreadLocalMap从而实现的ThreadLocal的存储。因此ThreadLocalMap可以称之为ThreadLocal的存储结构。ThreadLocalMap是ThreadLocal的内部类。 2、ThreadLocalMap内部是创建的Entry数组进行存储，Entry是一个弱引用，表明只要垃圾回收执行时，会回收掉它存储的对象。（至于为什么使用弱引用后文有阐述） 3、Thread中的threadLocals这个变量用于绑定一个ThreadLocalMap，而线程中所有创建的ThreadLocal 都会存储在这个threadLocals中。这样就保证了不同线程中的副本不会互相影响，因为ThreadLocalMap，即ThreadLocal的存储容器是与线程本身绑定的。 4、ThreadLocalMap如果发生冲突，会让index++，到下一个位置存取值，如果下一个位置还产生冲突，那么就继续index++。只有三种情况会跳出循环：
1. 该位置的Entry为空，那么就用key和value新建一个Entry赋值到这个位置 。1. 该位置的ThreadLocal为null，替换这个废弃的Entry。（一般情况就是由于弱引用被垃圾回收机制回收了）1. 该位置的ThreadLocal等于我们要赋值的ThreadLocal，直接使用value覆盖那个值。
# ThreadLocal初始化

```
    public ThreadLocal() {
    }

```

```
    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode =
        new AtomicInteger();
        
    private static final int HASH_INCREMENT = 0x61c88647;

    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

```

可以看到ThreadLocal默认的构造函数是空的，但是在ThreadLocal中却有初始化的内容。 threadLocalHashCode 顾名思义是一个用来保证ThreadLocal对象唯一性的一个int，我们一般称为hash，它仅仅会在ThreadLocal的内部类ThreadLocalMap中使用。

# ThreadLocalMap

>  
 笔者建议，要了解ThreadLocal的源码首先要了解HashMap源码，否则可能对ThreadLocalMap难以理解。 


它是ThreadLocal的内部静态类，用于保存线程中ThreadLocal的数据结构。ThreadLocal内部就是通过操作ThreadLocalMap从而实现的ThreadLocal的存储。

## ThreadLocalMap的初始化

```
        private static final int INITIAL_CAPACITY = 16;
        
        ThreadLocalMap(ThreadLocal firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode &amp; (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

```

```
        private void setThreshold(int len) {
            threshold = len * 2 / 3;
        }

```

它首先创建一个大小为16的实体数组，然后将threadLocalHashCode 和（length-1）进行与操作从而求得index，这个操作与threadLocalHashCode %length相等，之所以用与操作是因为更快。 获取到index之后，就把这第一个值存入数组。 至于设置的threshold ，其实就是代表着可用的length，如果超过了这个值，就会执行扩容操作。

>  
 对于“&amp;”操作和threshold ，阅读过HashMap源码的同学肯定深知其中原理，如果不解并且好奇者，可以看一下笔者的博客：  


## ThreadLocalMap.Entry（）

```
        static class Entry extends WeakReference&lt;ThreadLocal&gt; {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal k, Object v) {
                super(k);
                value = v;
            }
        }

```

这是ThreadLocalMap的内部类，ThreadLocalMap内部就是创建Entry 来保存的ThreadLocal。 这里要注意Entry 继承的弱引用，即只要垃圾回收执行，这个对象便会被回收。

### 为什么要使用弱引用呢？

笔者个人见解，在使用线程池的情况下，核心线程是不断复用不会被回收的，而ThreadLocalMap是与线程绑定的，因此会一直保留对ThreadLocalMap的引用。（即垃圾回收无法回收掉ThreadLocalMap） 如果不使用弱引用，那么在线程替换了一个Runnable之后，之前的Runnable用ThreadLocal存储的对象还会存在，并且不会被回收，这就发生了真正意义上的内存泄露，并且随着线程的复用次数的提高，ThreadLocalMap中的对象一直不会被回收，很可能会造成OOM。

## ThreadLocalMap.set（）

```
       private void set(ThreadLocal key, Object value) {

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode &amp; (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) &amp;&amp; sz &gt;= threshold)
                rehash();
        }

```

这里有一个循环，index会一直执行加1操作（但是小于length），有三种情况会停止：
1. 该位置的Entry为空，那么就用key和value新建一个Entry赋值到这个位置1. 到该位置的ThreadLocal为null，替换这个废弃的Entry。（一般情况就是由于弱引用被垃圾回收机制回收了）1. 到该位置的ThreadLocal等于我们要赋值的ThreadLocal，直接使用value覆盖那个值。
有些读者一定会惊讶，index++这是什么操作，为什么要这样处理？ 我们不得不去承认，冲突是无法完全避免。很有可能两个不同的ThreadLocal 是同一个index，那么这就是此处处理冲突的方式。

然后如果size大于了threshold（根据初始化中源码可知，threshold = len * 2 / 3），就会执行扩容操作。大致逻辑和HashMap的扩容操作相同，此处就不多加累述。如果好奇的同学可以自己查看源码或者阅读笔者关于HashMap的博文——。

## ThreadLocalMap.getEntry（）

这是ThreadLocalMap获取内部实例的操作。

```
        private Entry getEntry(ThreadLocal key) {
            int i = key.threadLocalHashCode &amp; (table.length - 1);
            Entry e = table[i];
            if (e != null &amp;&amp; e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }

```

正常情况下比较简单，如果table[i]不为null，并且该处的ThreadLocal 与导入的参数ThreadLocal 相等，就直接返回value。我们主要看一下非理想情况下的getEntryAfterMiss（）方法。

```
        private Entry getEntryAfterMiss(ThreadLocal key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                ThreadLocal k = e.get();
                if (k == key)
                    return e;
                if (k == null)
                    expungeStaleEntry(i);
                else
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }

```

还是会从i开始，执行遍历操作，如果table[i]处Entry的ThreadLocal 为空，表明可能Entry已经被回收或者本来就是设置的null，那么这个节点就没有意义，所以就删掉。如果table[i]处Entry的ThreadLocal 等于导入的参数ThreadLocal ，那么就返回这个值。

# ThreadLocal

ThreadLocalMap我们了解的差不多了。知道了ThreadLocal存储的数据结构，我们再去看ThreadLocal对其的操作就非常简单了。

# ThreadLocal.Set（）方法

```
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

```

首先获取到当前线程，然后通过当前线程获取到ThreadLocalMap ，如果ThreadLocalMap 不为null就将数据存进去，如果为null就创建ThreadLocalMap 。 我们先看一下getMap（）方法。

```
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

```

```
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

```

它返回的是Thread.threadLocals这个变量，每个Thread都会都一个这样的变量，用于存储附属于它的ThreadLocalMap 。而线程中所有创建的ThreadLocal 都会存储在这个threadLocals中。这样就保证了不同线程中的副本不会互相影响，因为ThreadLocalMap，即ThreadLocal的存储容器是与线程本身绑定的。

# ThreadLocal.get（）方法

```
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue();
    }


```

首先还是通过线程获取到ThreadLocalMap 。然后获取到ThreadLocalMap 中的Entry ，再获取Entry 中的value。如果获取不到ThreadLocalMap 的话，就会初始化一个value返回。setInitialValue()逻辑比较简单，我可以先看一下它的源码。

```
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

```

```
    protected T initialValue() {
        return null;
    }

```

根据initialValue()这个方法进行value的初始化，但是默认情况下是null。然后会创建ThreadLocalMap ，并且把这个初始化的值放进去。 那么既然默认返回值是null，又何必要有这个方法呢？我们可以看一下initialValue()的前缀是protected ，所以说这个方法是为了继承重写用的。我们一般情况也用不到。

>  
 ThreadLocal知识点比较杂乱，笔者可能有地方没有讲清楚，欢迎读者留言提醒。 

