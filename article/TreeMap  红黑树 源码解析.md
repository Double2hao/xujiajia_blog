#TreeMap  红黑树 源码解析
# 概念

红黑树是一种**平衡二叉搜索树**。它可以在O(log n)的时间内完成查找，插入和删除。

**二叉搜索树：** 左边的节点都小于父节点，右边的节点都大于父节点。 **平衡二叉树：** 任意左右两个子树的叶子节点的高度相差不超过1。

>  
 关于二叉树的分类可以看笔者的这篇文章： 


分成这两个概念来看，红黑树就非常好理解了。
1. 二叉搜索树的特性非常容易满足，每次在添加新节点的时候如果小于当前节点，就放到左边，如果大于当前节点就放到右边。1. 红黑树为了满足平衡二叉树的特性，主要是通过“左旋”和“右旋”两个操作。 那么在什么场景下用“左旋”，什么场景下用“右旋”呢？这就需要判断当前节点是“红”还是“黑”了。也是由于红黑树的节点有红黑之分，所以被称为“红黑树”。
# 左旋和右旋

### 左旋demo图和源码

<img src="https://img-blog.csdnimg.cn/20190822165227652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

```
    private void rotateLeft(Entry&lt;K,V&gt; p) {
        if (p != null) {
            Entry&lt;K,V&gt; r = p.right;
            p.right = r.left;
            if (r.left != null)
                r.left.parent = p;
            r.parent = p.parent;
            if (p.parent == null)
                root = r;
            else if (p.parent.left == p)
                p.parent.left = r;
            else
                p.parent.right = r;
            r.left = p;
            p.parent = r;
        }
    }

```

### 右旋demo图和源码

<img src="https://img-blog.csdnimg.cn/20190822165257626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

```
    private void rotateRight(Entry&lt;K,V&gt; p) {
        if (p != null) {
            Entry&lt;K,V&gt; l = p.left;
            p.left = l.right;
            if (l.right != null) l.right.parent = p;
            l.parent = p.parent;
            if (p.parent == null)
                root = l;
            else if (p.parent.right == p)
                p.parent.right = l;
            else p.parent.left = l;
            l.right = p;
            p.parent = l;
        }
    }

```

# 源码分析

红黑树的查找操作完全与二叉搜索树相同，因此此处就不增长篇幅分析了。 本文主要分析添加与删除操作的源码。

>  
 本文对红黑树"调整结构"部分的源码没有分析。 笔者也花了较多时间在分析该块代码上，虽然在源码上该块代码并不多，但是其判断“左旋”、“右旋”、“设置颜色”的条件没法通过上下文关系理解。 个人想来应该是是1978年的红黑树论文中论证了通过这样的条件可以保证红黑树的平衡性。对于一般的源码阅读者，就像记住定理一样，知道如此能保证红黑树的平衡性就可以了。 . 笔者想，与其“不知其所以然”地将代码在文章中复述一遍，不如就不讲了。 如果有读者对这一块比较熟悉的，欢迎评论交流。 


## 添加源码

主要逻辑：
1. 直接当做二叉搜索树来做插入逻辑。 如果值比当前node小，就走左边，如果比当前node大就走右边。1. 在做完插入之后，通过fixAfterInsertion()这个方法来做红黑树的结构修正。1. fixAfterInsertion()中主要有两个逻辑，一是通过左旋右旋让红黑树到达平衡状态，二是重新为各节点附上红黑的颜色。
```
    public V put(K key, V value) {
        TreeMapEntry&lt;K,V&gt; t = root;
        if (t == null) {
            if (comparator != null) {
                if (key == null) {
                    comparator.compare(key, key);
                }
            } else {
                if (key == null) {
                    throw new NullPointerException("key == null");
                } else if (!(key instanceof Comparable)) {
                    throw new ClassCastException(
                            "Cannot cast" + key.getClass().getName() + " to Comparable.");
                }
            }
            // END Android-changed: Work around buggy comparators. http://b/34084348
            root = new TreeMapEntry&lt;&gt;(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        TreeMapEntry&lt;K,V&gt; parent;
        // split comparator and comparable paths
        Comparator&lt;? super K&gt; cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp &lt; 0)
                    t = t.left;
                else if (cmp &gt; 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable&lt;? super K&gt; k = (Comparable&lt;? super K&gt;) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp &lt; 0)
                    t = t.left;
                else if (cmp &gt; 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        TreeMapEntry&lt;K,V&gt; e = new TreeMapEntry&lt;&gt;(key, value, parent);
        if (cmp &lt; 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }

```

## 删除源码

主要逻辑：
1. 先删除在二叉树中的结点。1. 删除该结点后通过fixAfterDeletion()来调整红黑树的结构。1. fixAfterDeletion()中主要有两个逻辑，一是通过左旋右旋让红黑树到达平衡状态，二是重新为各节点附上红黑的颜色。
删除二叉树中的结点有三种情况：
1. 如果该结点是叶子结点，那么直接把该结点从父结点删除。1. 如果该结点只有左孩子，那么让他 的左孩子来替代它原来的位置。1. 如果该结点只有右孩子，那么让他 的右孩子来替代它原来的位置。1. 如果该结点左右都有孩子，那么该结点会选择右子树中最小的值来替换自己的值，然后把右子树中最小的值删除。
```
    public V remove(Object key) {
        TreeMapEntry&lt;K,V&gt; p = getEntry(key);
        if (p == null)
            return null;

        V oldValue = p.value;
        deleteEntry(p);
        return oldValue;
    }

```

```
    private void deleteEntry(TreeMapEntry&lt;K,V&gt; p) {
        modCount++;
        size--;

        // If strictly internal, copy successor's element to p and then make p
        // point to successor.
        if (p.left != null &amp;&amp; p.right != null) {
            TreeMapEntry&lt;K,V&gt; s = successor(p);
            p.key = s.key;
            p.value = s.value;
            p = s;
        } // p has 2 children

        // Start fixup at replacement node, if it exists.
        TreeMapEntry&lt;K,V&gt; replacement = (p.left != null ? p.left : p.right);

        if (replacement != null) {
            // Link replacement to parent
            replacement.parent = p.parent;
            if (p.parent == null)
                root = replacement;
            else if (p == p.parent.left)
                p.parent.left  = replacement;
            else
                p.parent.right = replacement;

            // Null out links so they are OK to use by fixAfterDeletion.
            p.left = p.right = p.parent = null;

            // Fix replacement
            if (p.color == BLACK)
                fixAfterDeletion(replacement);
        } else if (p.parent == null) { // return if we are the only node.
            root = null;
        } else { //  No children. Use self as phantom replacement and unlink.
            if (p.color == BLACK)
                fixAfterDeletion(p);

            if (p.parent != null) {
                if (p == p.parent.left)
                    p.parent.left = null;
                else if (p == p.parent.right)
                    p.parent.right = null;
                p.parent = null;
            }
        }
    }

```

```
    static &lt;K,V&gt; TreeMapEntry&lt;K,V&gt; successor(TreeMapEntry&lt;K,V&gt; t) {
        if (t == null)
            return null;
        else if (t.right != null) {
            TreeMapEntry&lt;K,V&gt; p = t.right;
            while (p.left != null)
                p = p.left;
            return p;
        } else {
            TreeMapEntry&lt;K,V&gt; p = t.parent;
            TreeMapEntry&lt;K,V&gt; ch = t;
            while (p != null &amp;&amp; ch == p.right) {
                ch = p;
                p = p.parent;
            }
            return p;
        }
    }

```