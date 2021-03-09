#SparseArray 源码解析
# 前言

在Android开发中，在key为Integer的情况下，都建议不用HashMap，使用SparseArray替换。

SparseArray与HashMap相比究竟有什么好处呢？

# 概要

个人认为两者主要有以下几点区别：
1. SparseArray 比HashMap更轻量，更节省内存。1. SparseArray的速度肯定比HashMap慢，但是在数据不多的时候，这点速度可以忽略不计。
# 源码解析

## 构造

SpaseArray使用两个数组来存储key和value。 构造主要有两种情况：
1. 如果capacity=0，直接初始化两个length为0的数组。1. 如果capacity&gt;0，就使用ArrayUtils初始一个可以增长的value数组，然后初始化定长的key数组。
```
	private int[] mKeys;
    private Object[] mValues;
    private int mSize;
    
    public SparseArray() {
        this(10);
    }

    public SparseArray(int initialCapacity) {
        if (initialCapacity == 0) {
            mKeys = EmptyArray.INT;
            mValues = EmptyArray.OBJECT;
        } else {
            mValues = ArrayUtils.newUnpaddedObjectArray(initialCapacity);
            mKeys = new int[mValues.length];
        }
        mSize = 0;
    }

```

```
final class EmptyArray {
    private EmptyArray() {}

    static final boolean[] BOOLEAN = new boolean[0];
    static final byte[] BYTE = new byte[0];
    static final char[] CHAR = new char[0];
    static final double[] DOUBLE = new double[0];
    static final int[] INT = new int[0];

    static final Class&lt;?&gt;[] CLASS = new Class&lt;?&gt;[ 0 ];
    static final Object[] OBJECT = new Object[0];
    static final String[] STRING = new String[0];
    static final Throwable[] THROWABLE = new Throwable[0];
    static final StackTraceElement[] STACK_TRACE_ELEMENT = new StackTraceElement[0];
}

```

```
ArrayUtils
    @SuppressWarnings("unchecked")
    public static &lt;T&gt; T[] newUnpaddedArray(Class&lt;T&gt; clazz, int minLen) {
        return (T[])VMRuntime.getRuntime().newUnpaddedArray(clazz, minLen);
    }

```

```
VMRuntime
    /**
     * Returns an array of at least minLength, but potentially larger. The increased size comes from
     * avoiding any padding after the array. The amount of padding varies depending on the
     * componentType and the memory allocator implementation.
     */
    public native Object newUnpaddedArray(Class&lt;?&gt; componentType, int minLength);


```

## get()

使用二分查找在有序的mKeys数组中找到value的位置，然后输出value。

```
    public E get(int key) {
        return get(key, null);
    }

    @SuppressWarnings("unchecked")
    public E get(int key, E valueIfKeyNotFound) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

        if (i &lt; 0 || mValues[i] == DELETED) {
            return valueIfKeyNotFound;
        } else {
            return (E) mValues[i];
        }
    }

```

```
ContainerHelpers
    static int binarySearch(int[] array, int size, int value) {
        int lo = 0;
        int hi = size - 1;

        while (lo &lt;= hi) {
            final int mid = (lo + hi) &gt;&gt;&gt; 1;
            final int midVal = array[mid];

            if (midVal &lt; value) {
                lo = mid + 1;
            } else if (midVal &gt; value) {
                hi = mid - 1;
            } else {
                return mid;  // value found
            }
        }
        return ~lo;  // value not present
    }

```

## delete()

在删除的时候会在原来的value的位置用DELETED这个object值代替，然后再下次gc的时候将数据给回收掉。 gc逻辑将会在put中触发。

```
    private static final Object DELETED = new Object();

    public void delete(int key) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

        if (i &gt;= 0) {
            if (mValues[i] != DELETED) {
                mValues[i] = DELETED;
                mGarbage = true;
            }
        }
    }

    public E removeReturnOld(int key) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

        if (i &gt;= 0) {
            if (mValues[i] != DELETED) {
                final E old = (E) mValues[i];
                mValues[i] = DELETED;
                mGarbage = true;
                return old;
            }
        }
        return null;
    }

    public void remove(int key) {
        delete(key);
    }

    public void removeAt(int index) {
        if (mValues[index] != DELETED) {
            mValues[index] = DELETED;
            mGarbage = true;
        }
    }

    public void removeAtRange(int index, int size) {
        final int end = Math.min(mSize, index + size);
        for (int i = index; i &lt; end; i++) {
            removeAt(i);
        }
    }

```

## put()

put流程如下：
1. 首先通过二分查找要插入的位置1. 如果已经存在就覆盖，如果不存在就新插入。1. 判断是否需要GC，如果需要就触发gc逻辑1. 如果插入的位置超过了size，那么就使用GrowingArrayUtils扩容。扩容就是生成一个新的size*2的数组，然后将原来的内容复制过去。
```
    public void put(int key, E value) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

        if (i &gt;= 0) {
            mValues[i] = value;
        } else {
            i = ~i;

            if (i &lt; mSize &amp;&amp; mValues[i] == DELETED) {
                mKeys[i] = key;
                mValues[i] = value;
                return;
            }

            if (mGarbage &amp;&amp; mSize &gt;= mKeys.length) {
                gc();

                // Search again because indices may have changed.
                i = ~ContainerHelpers.binarySearch(mKeys, mSize, key);
            }

            mKeys = GrowingArrayUtils.insert(mKeys, mSize, i, key);
            mValues = GrowingArrayUtils.insert(mValues, mSize, i, value);
            mSize++;
        }
    }

```

```
GrowingArrayUtils

    public static &lt;T&gt; T[] insert(T[] array, int currentSize, int index, T element) {
        assert currentSize &lt;= array.length;

        if (currentSize + 1 &lt;= array.length) {
            System.arraycopy(array, index, array, index + 1, currentSize - index);
            array[index] = element;
            return array;
        }

        @SuppressWarnings("unchecked")
        T[] newArray = ArrayUtils.newUnpaddedArray((Class&lt;T&gt;)array.getClass().getComponentType(),
                growSize(currentSize));
        System.arraycopy(array, 0, newArray, 0, index);
        newArray[index] = element;
        System.arraycopy(array, index, newArray, index + 1, array.length - index);
        return newArray;
    }

    /**
     * Primitive int version of {@link #insert(Object[], int, int, Object)}.
     */
    public static int[] insert(int[] array, int currentSize, int index, int element) {
        assert currentSize &lt;= array.length;

        if (currentSize + 1 &lt;= array.length) {
            System.arraycopy(array, index, array, index + 1, currentSize - index);
            array[index] = element;
            return array;
        }

        int[] newArray = ArrayUtils.newUnpaddedIntArray(growSize(currentSize));
        System.arraycopy(array, 0, newArray, 0, index);
        newArray[index] = element;
        System.arraycopy(array, index, newArray, index + 1, array.length - index);
        return newArray;
    }

    public static int growSize(int currentSize) {
        return currentSize &lt;= 4 ? 8 : currentSize * 2;
    }

```

## gc()

gc逻辑很简单，将数组中所有非DELETED的对象向前挪，这样前面的DELETED就会被覆盖掉。 新的size只会统计原来数组中非DELETED的对象，这样对实现了逻辑删除。

```
    private void gc() {

        int n = mSize;
        int o = 0;
        int[] keys = mKeys;
        Object[] values = mValues;

        for (int i = 0; i &lt; n; i++) {
            Object val = values[i];

            if (val != DELETED) {
                if (i != o) {
                    keys[o] = keys[i];
                    values[o] = val;
                    values[i] = null;
                }

                o++;
            }
        }

        mGarbage = false;
        mSize = o;
    }

```
