#HashMap源码分析
# 前提概要：

HashMap算是Map的子类中用的最多的了，其相关的类也有很多，比如LinkedHashMap、HashMap、HashTable、ConcurrentHashMap。而HashMap在其中也扮演着一个很重要的角色，本文主要分析HashMap的源码，了解完后，其他的也自然触类旁通了。

# 概述

HashMap本身是数组+链表的存储结构（JDK1.8加上了红黑树部分，本篇文章不提及，仅仅在目前android所使用的HashMap版本上进行讲述）。

put的大致流程如下： 1、通过hashcode计算出key的hash值 2、通过hash%length计算出存储在table中的index（源码中是使用hash&amp;(length-1)，这样结果相同，但是更快） 3、如果此时table[index]的值为空，那么就直接存储，如果不为空那么就链接到这个数所在的链表的头部。（在JDK1.8中，如果链表长度大于8就转化成红黑树）

get的大致流程如下： 1、通过hashcode计算出key的hash值 2、通过hash%length计算出存储在table中的index（源码中是使用hash&amp;(length-1)，这样结果相同，但是更快） 3、遍历table[index]所在的链表，只有当key与该节点中的key的值相同时才取出。

# 主要变量

```
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
        static final HashMapEntry&lt;?,?&gt;[] EMPTY_TABLE = {};
    transient HashMapEntry&lt;K,V&gt;[] table = (HashMapEntry&lt;K,V&gt;[]) EMPTY_TABLE;
    transient int size;
    int threshold;
    final float loadFactor = DEFAULT_LOAD_FACTOR;

```

**table**:是一个HashMapEntry的数组，同时也是表明HashMap本身的数据结构是数组。一开始是为空的。 **size**:整个HashMap中的键值对的个数。 **loadFactor**:负载因子，默认为0.75。表明如果使用到的容量达到了四分之三，那么就进行扩容操作。 **threshold**:HashMap所能容纳的最大数据量的HashMapEntry(键值对)个数。其值一般情况下等于loadFactor *table.length。如果size&gt;threshold,那么就进行扩容操作。

# HashMapEntry

HashMapEntry是存在HashMap的数组中所存的数据结构,也是HashMap的内部类，源码如下：

```
   static class HashMapEntry&lt;K,V&gt; implements Map.Entry&lt;K,V&gt; {
        final K key;
        V value;
        HashMapEntry&lt;K,V&gt; next;
        int hash;

        HashMapEntry(int h, K k, V v, HashMapEntry&lt;K,V&gt; n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null &amp;&amp; k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null &amp;&amp; v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }


        void recordAccess(HashMap&lt;K,V&gt; m) {
        }


        void recordRemoval(HashMap&lt;K,V&gt; m) {
        }
    }

```

可以看到结构其实比较简单，其中存key，value，hash以及链接的下一个HashMapEntry。这也就是数组+链表的数据结构的真身。

# 构造函数

```
    static final int DEFAULT_INITIAL_CAPACITY = 4;
    static final int MAXIMUM_CAPACITY = 1 &lt;&lt; 30;
    
   public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity &lt; 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity &gt; MAXIMUM_CAPACITY) {
            initialCapacity = MAXIMUM_CAPACITY;
        } else if (initialCapacity &lt; DEFAULT_INITIAL_CAPACITY) {
            initialCapacity = DEFAULT_INITIAL_CAPACITY;
        }

        if (loadFactor &lt;= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        threshold = initialCapacity;
        init();
    }
    
    void init() {
    }

```

此处主要是初始化了threshold ，规定其范围在4~2^30之间。如果小于就等于最小值，如果大于就大于最大值。 此处没有初始化table数组，说明在put之前是一直为空的。

# put操作

```
    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = sun.misc.Hashing.singleWordWangJenkinsHash(key);
        int i = indexFor(hash, table.length);
        for (HashMapEntry&lt;K,V&gt; e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash &amp;&amp; ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }

```

```
		void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size &gt;= threshold) &amp;&amp; (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? sun.misc.Hashing.singleWordWangJenkinsHash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }

```

```
        void createEntry(int hash, K key, V value, int bucketIndex) {
        HashMapEntry&lt;K,V&gt; e = table[bucketIndex];
        table[bucketIndex] = new HashMapEntry&lt;&gt;(hash, key, value, e);
        size++;
    }

```

整个put的流程还是比较简单，首先遍历计算hash值和index，然后遍历该处链表，如果已存在该key就替换value。如果该key不存在，那么就直接用addEntry（）在该处创建一个HashMapEntry，如果table[index]已经存在其他键值对，那么就把原来的键值对作为next链接到现在创建的HashMapEntry尾部。

此处比较关键的还是table为空的情况，主要是inflateTable函数：

```
private void inflateTable(int toSize) {
        // Find a power of 2 &gt;= toSize
        int capacity = roundUpToPowerOf2(toSize);

        // Android-changed: Replace usage of Math.min() here because this method is
        // called from the &lt;clinit&gt; of runtime, at which point the native libraries
        // needed by Float.* might not be loaded.
        float thresholdFloat = capacity * loadFactor;
        if (thresholdFloat &gt; MAXIMUM_CAPACITY + 1) {
            thresholdFloat = MAXIMUM_CAPACITY + 1;
        }

        threshold = (int) thresholdFloat;
        table = new HashMapEntry[capacity];
    }

```

首先通过roundUpToPowerOf2(toSize)计算出容量，roundUpToPowerOf2（toSize）的作用是计算出小于等于toSize的2的幂次方。然后再根据容量新建table。 此处就有一个HashMap的关键点，**为什么HashMap的容量要等于2的 幂次方？** 此处还是要提到table的index的计算方式，计算的时候是用hash&amp;（length-1）来替代hash%length，而只有在length为2的幂次方的时候两者才会相等。 #get操作:

```
    public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry&lt;K,V&gt; entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }

```

```
   final Entry&lt;K,V&gt; getEntry(Object key) {
        if (size == 0) {
            return null;
        }

        int hash = (key == null) ? 0 : sun.misc.Hashing.singleWordWangJenkinsHash(key);
        for (HashMapEntry&lt;K,V&gt; e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &amp;&amp;
                ((k = e.key) == key || (key != null &amp;&amp; key.equals(k))))
                return e;
        }
        return null;
    }

```

get操作比较简单：遍历table[index]处的链表，如果key等于该处的HashMapEntry的key那么就取出该值。 #resize()操作 一般操作中，resize（）是在put操作的addEntry（）中才会执行，也就是说在给table[index]新建实例的时候才会执行resize（）操作。条件可以看下源码：

```
    void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size &gt;= threshold) &amp;&amp; (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? sun.misc.Hashing.singleWordWangJenkinsHash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }

```

当size&gt;threshold的时候就执行resize操作，新的容量是原来的两倍。

```
    void resize(int newCapacity) {
        HashMapEntry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        HashMapEntry[] newTable = new HashMapEntry[newCapacity];
        transfer(newTable);
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }

```

```
    void transfer(HashMapEntry[] newTable) {
        int newCapacity = newTable.length;
        for (HashMapEntry&lt;K,V&gt; e : table) {
            while(null != e) {
                HashMapEntry&lt;K,V&gt; next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }

```

resize中首先根据新的容量创建一个HashMapEntry的数组，然后交给transfer()进行数据的转移。 transfer()中遍历旧的数组，根据hash值和新的容量计算出新的index，然后放入新的数组。
