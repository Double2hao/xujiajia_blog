#RecyclerView 回收机制解析
# 概述

近期接触到RecyclerView回收机制相关的内容，于是作此文记录下相关探索。 如记录的有问题欢迎评论探讨。

# 定位回收机制源码

笔者使用debug的方式，在adapter的onCreateViewHolder中打断点，于是得到该任务栈。

>  
 思路：onCreateViewHolder是创建viewHolder的方法，那么在调用这个方法之前一定有是否复用的判断，因此断点在这个方法可以找到相关方法。 


然后我们就可以发现是RecyclerView#Recycler的tryGetViewHolderForPositionByDeadline()方法中处理的相关的回收逻辑，源码如下：

```
        ViewHolder tryGetViewHolderForPositionByDeadline(int position,
                boolean dryRun, long deadlineNs) {<!-- -->
            if (position &lt; 0 || position &gt;= mState.getItemCount()) {<!-- -->
                throw new IndexOutOfBoundsException("Invalid item position " + position
                        + "(" + position + "). Item count:" + mState.getItemCount()
                        + exceptionLabel());
            }
            boolean fromScrapOrHiddenOrCache = false;
            ViewHolder holder = null;
            // 0) If there is a changed scrap, try to find from there
            if (mState.isPreLayout()) {<!-- -->
                holder = getChangedScrapViewForPosition(position);
                fromScrapOrHiddenOrCache = holder != null;
            }
            // 1) Find by position from scrap/hidden list/cache
            if (holder == null) {<!-- -->
                holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun);
                if (holder != null) {<!-- -->
                    if (!validateViewHolderForOffsetPosition(holder)) {<!-- -->
                        // recycle holder (and unscrap if relevant) since it can't be used
                        if (!dryRun) {<!-- -->
                            // we would like to recycle this but need to make sure it is not used by
                            // animation logic etc.
                            holder.addFlags(ViewHolder.FLAG_INVALID);
                            if (holder.isScrap()) {<!-- -->
                                removeDetachedView(holder.itemView, false);
                                holder.unScrap();
                            } else if (holder.wasReturnedFromScrap()) {<!-- -->
                                holder.clearReturnedFromScrapFlag();
                            }
                            recycleViewHolderInternal(holder);
                        }
                        holder = null;
                    } else {<!-- -->
                        fromScrapOrHiddenOrCache = true;
                    }
                }
            }
            if (holder == null) {<!-- -->
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                if (offsetPosition &lt; 0 || offsetPosition &gt;= mAdapter.getItemCount()) {<!-- -->
                    throw new IndexOutOfBoundsException("Inconsistency detected. Invalid item "
                            + "position " + position + "(offset:" + offsetPosition + ")."
                            + "state:" + mState.getItemCount() + exceptionLabel());
                }

                final int type = mAdapter.getItemViewType(offsetPosition);
                // 2) Find from scrap/cache via stable ids, if exists
                if (mAdapter.hasStableIds()) {<!-- -->
                    holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition),
                            type, dryRun);
                    if (holder != null) {<!-- -->
                        // update position
                        holder.mPosition = offsetPosition;
                        fromScrapOrHiddenOrCache = true;
                    }
                }
                if (holder == null &amp;&amp; mViewCacheExtension != null) {<!-- -->
                    // We are NOT sending the offsetPosition because LayoutManager does not
                    // know it.
                    final View view = mViewCacheExtension
                            .getViewForPositionAndType(this, position, type);
                    if (view != null) {<!-- -->
                        holder = getChildViewHolder(view);
                        if (holder == null) {<!-- -->
                            throw new IllegalArgumentException("getViewForPositionAndType returned"
                                    + " a view which does not have a ViewHolder"
                                    + exceptionLabel());
                        } else if (holder.shouldIgnore()) {<!-- -->
                            throw new IllegalArgumentException("getViewForPositionAndType returned"
                                    + " a view that is ignored. You must call stopIgnoring before"
                                    + " returning this view." + exceptionLabel());
                        }
                    }
                }
                if (holder == null) {<!-- --> // fallback to pool
                    if (DEBUG) {<!-- -->
                        Log.d(TAG, "tryGetViewHolderForPositionByDeadline("
                                + position + ") fetching from shared pool");
                    }
                    holder = getRecycledViewPool().getRecycledView(type);
                    if (holder != null) {<!-- -->
                        holder.resetInternal();
                        if (FORCE_INVALIDATE_DISPLAY_LIST) {<!-- -->
                            invalidateDisplayListInt(holder);
                        }
                    }
                }
                if (holder == null) {<!-- -->
                    long start = getNanoTime();
                    if (deadlineNs != FOREVER_NS
                            &amp;&amp; !mRecyclerPool.willCreateInTime(type, start, deadlineNs)) {<!-- -->
                        // abort - we have a deadline we can't meet
                        return null;
                    }
                    holder = mAdapter.createViewHolder(RecyclerView.this, type);
                    if (ALLOW_THREAD_GAP_WORK) {<!-- -->
                        // only bother finding nested RV if prefetching
                        RecyclerView innerView = findNestedRecyclerView(holder.itemView);
                        if (innerView != null) {<!-- -->
                            holder.mNestedRecyclerView = new WeakReference&lt;&gt;(innerView);
                        }
                    }

                    long end = getNanoTime();
                    mRecyclerPool.factorInCreateTime(type, end - start);
                    if (DEBUG) {<!-- -->
                        Log.d(TAG, "tryGetViewHolderForPositionByDeadline created new ViewHolder");
                    }
                }
            }

            // This is very ugly but the only place we can grab this information
            // before the View is rebound and returned to the LayoutManager for post layout ops.
            // We don't need this in pre-layout since the VH is not updated by the LM.
            if (fromScrapOrHiddenOrCache &amp;&amp; !mState.isPreLayout() &amp;&amp; holder
                    .hasAnyOfTheFlags(ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST)) {<!-- -->
                holder.setFlags(0, ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);
                if (mState.mRunSimpleAnimations) {<!-- -->
                    int changeFlags = ItemAnimator
                            .buildAdapterChangeFlagsForAnimations(holder);
                    changeFlags |= ItemAnimator.FLAG_APPEARED_IN_PRE_LAYOUT;
                    final ItemHolderInfo info = mItemAnimator.recordPreLayoutInformation(mState,
                            holder, changeFlags, holder.getUnmodifiedPayloads());
                    recordAnimationInfoIfBouncedHiddenView(holder, info);
                }
            }

            boolean bound = false;
            if (mState.isPreLayout() &amp;&amp; holder.isBound()) {<!-- -->
                // do not update unless we absolutely have to.
                holder.mPreLayoutPosition = position;
            } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {<!-- -->
                if (DEBUG &amp;&amp; holder.isRemoved()) {<!-- -->
                    throw new IllegalStateException("Removed holder should be bound and it should"
                            + " come here only in pre-layout. Holder: " + holder
                            + exceptionLabel());
                }
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs);
            }

            final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
            final LayoutParams rvLayoutParams;
            if (lp == null) {<!-- -->
                rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else if (!checkLayoutParams(lp)) {<!-- -->
                rvLayoutParams = (LayoutParams) generateLayoutParams(lp);
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else {<!-- -->
                rvLayoutParams = (LayoutParams) lp;
            }
            rvLayoutParams.mViewHolder = holder;
            rvLayoutParams.mPendingInvalidate = fromScrapOrHiddenOrCache &amp;&amp; bound;
            return holder;
        }

```

# 回收机制

tryGetViewHolderForPositionByDeadline()源码中有几个获得ViewHodler的方式：

>  
 - holder = getChangedScrapViewForPosition(position);- holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun);- final View view = mViewCacheExtension .getViewForPositionAndType(this, position, type);- holder = getRecycledViewPool().getRecycledView(type);- holder = mAdapter.createViewHolder(RecyclerView.this, type); 


笔者整理后，根据RecyclerView#Recycler的几个参数，按照代码顺序是如下几个回收模块：
1. mAttachedScrap1. mChangedScrap1. mCachedViews1. mViewCacheExtension1. mRecyclerPool
# mAttachedScrap与mChangedScrap
- mAttachedScrap 用于存储ViewHolder已经从RecyclerView上移除，但是仍有可能被复用的View。- mChangedScrap 用于存储ViewHolder仍在RecyclerView上，但是数据已经过时，需要被更新的View。
>  
 针对这两种场景，笔者特别做了debug实验： 
 - mChangedScrap： 场景一：通过adapter设置notifyItemChanged（index），如果当前index显示在屏幕中，这个index的ViewHodler会被存储到mChangedScrap中。- mAttachedScrap： 场景一：让一个RecyclerView 隐藏后再显示，当前页面的中的item都会存储到mAttachedScrap中。 场景二：通过adapter设置notifyItemChanged（index），当前屏幕中index以外的其他item都会被存储到mAttachedScrap中。 


## 源码解析

在源码中查找mAttachedScrap与mChangedScrap填充内容的地方，可以找到是在同一处。

```
        /**
         * Mark an attached view as scrap.
         *
         * &lt;p&gt;"Scrap" views are still attached to their parent RecyclerView but are eligible
         * for rebinding and reuse. Requests for a view for a given position may return a
         * reused or rebound scrap view instance.&lt;/p&gt;
         *
         * @param view View to scrap
         */
        void scrapView(View view) {<!-- -->
            final ViewHolder holder = getChildViewHolderInt(view);
            if (holder.hasAnyOfTheFlags(ViewHolder.FLAG_REMOVED | ViewHolder.FLAG_INVALID)
                    || !holder.isUpdated() || canReuseUpdatedViewHolder(holder)) {<!-- -->
                if (holder.isInvalid() &amp;&amp; !holder.isRemoved() &amp;&amp; !mAdapter.hasStableIds()) {<!-- -->
                    throw new IllegalArgumentException("Called scrap view with an invalid view."
                            + " Invalid views cannot be reused from scrap, they should rebound from"
                            + " recycler pool." + exceptionLabel());
                }
                holder.setScrapContainer(this, false);
                mAttachedScrap.add(holder);
            } else {<!-- -->
                if (mChangedScrap == null) {<!-- -->
                    mChangedScrap = new ArrayList&lt;ViewHolder&gt;();
                }
                holder.setScrapContainer(this, true);
                mChangedScrap.add(holder);
            }
        }
        

```

通过上述的注释可知，这两个list都是用来存储不需要的view，然后后面准备复用的。 从源码中可以看到，需要ViewHolder本身没有失效（postsition,id,viewType都有效），否则会报异常。 另外的，两个list分别会存储ViewHolder的场景如下。

### mAttachedScrap

由源码if-else中可知，mAttachedScrap有三种场景会进入：
1. ViewHolder有ViewHolder.FLAG_REMOVED1. ViewHolder中没有ViewHolder.FLAG_REMOVED和ViewHolder.FLAG_UPDATE1. ViewHolder中没有ViewHolder.FLAG_REMOVED，但是有ViewHolder.FLAG_UPDATE，并且canReuseUpdatedViewHolder(holder)返回true
### mChangedScrap

其他的场景都会存储到mChangedScrap中，需要同时满足这几个条件:
1. ViewHolder中没有ViewHolder.FLAG_REMOVED1. ViewHolder有ViewHolder.FLAG_UPDATE1. canReuseUpdatedViewHolder(holder)返回false
### ViewHolder的几个Flag

根据上面的面源码，根据if-else里的判断条件，可以看到两个的区别首先在三个flag上，FLAG_REMOVED，FLAG_INVALID，FLAG_UPDATE。 这三个flag的源码和注释如下：

```
        /**
         * The data this ViewHolder's view reflects is stale and needs to be rebound
         * by the adapter. mPosition and mItemId are consistent.
         */
        static final int FLAG_UPDATE = 1 &lt;&lt; 1;
        
                /**
         * This ViewHolder's data is invalid. The identity implied by mPosition and mItemId
         * are not to be trusted and may no longer match the item view type.
         * This ViewHolder must be fully rebound to different data.
         */
        static final int FLAG_INVALID = 1 &lt;&lt; 2;

        /**
         * This ViewHolder points at data that represents an item previously removed from the
         * data set. Its view may still be used for things like outgoing animations.
         */
        static final int FLAG_REMOVED = 1 &lt;&lt; 3;

```

翻译后，三个flag的描述大概如下：
- FLAG_UPDATE 这个ViewHolder需要通过adapter来bind内容。这个ViewHolder的postion和id是统一的，有效的。- FLAG_REMOVED ViewHolder已经过时了，但是这个view仍可能被复用，比如用在跳出的动画。- FLAG_INVALID 这个ViewHolder中的数据已经无效了。也无法通过postion或者id来识别这个ViewHolder（position和id无效了），也不会与这个item的viewType匹配。 这个ViewHolder会完全与其他的数据绑定。
### canReuseUpdatedViewHolder

```
    boolean canReuseUpdatedViewHolder(ViewHolder viewHolder) {<!-- -->
        return mItemAnimator == null || mItemAnimator.canReuseUpdatedViewHolder(viewHolder,
                viewHolder.getUnmodifiedPayloads());
    }
    
    

```

```
/**
         * When an item is changed, ItemAnimator can decide whether it wants to re-use
         * the same ViewHolder for animations or RecyclerView should create a copy of the
         * item and ItemAnimator will use both to run the animation (e.g. cross-fade).
         * &lt;p&gt;
         * Note that this method will only be called if the {@link ViewHolder} still has the same
         * type ({@link Adapter#getItemViewType(int)}). Otherwise, ItemAnimator will always receive
         * both {@link ViewHolder}s in the
         * {@link #animateChange(ViewHolder, ViewHolder, ItemHolderInfo, ItemHolderInfo)} method.
         *
         * @param viewHolder The ViewHolder which represents the changed item's old content.
         * @param payloads A non-null list of merged payloads that were sent with change
         *                 notifications. Can be empty if the adapter is invalidated via
         *                 {@link RecyclerView.Adapter#notifyDataSetChanged()}. The same list of
         *                 payloads will be passed into
         *                 {@link RecyclerView.Adapter#onBindViewHolder(ViewHolder, int, List)}
         *                 method &lt;b&gt;if&lt;/b&gt; this method returns &lt;code&gt;true&lt;/code&gt;.
         *
         * @return True if RecyclerView should just rebind to the same ViewHolder or false if
         *         RecyclerView should create a new ViewHolder and pass this ViewHolder to the
         *         ItemAnimator to animate. Default implementation calls
         *         {@link #canReuseUpdatedViewHolder(ViewHolder)}.
         *
         * @see #canReuseUpdatedViewHolder(ViewHolder)
         */
        public boolean canReuseUpdatedViewHolder(@NonNull ViewHolder viewHolder,
                @NonNull List&lt;Object&gt; payloads) {<!-- -->
            return canReuseUpdatedViewHolder(viewHolder);
        }

```

根据源码可知，这个方法的作用如下：
1. 首先判断item有无动画，如果没有动画直接返回true。1. 判断item的动画能否复用ViewHolder来播放动画，还是需要新建一个ViewHolder。
# mCachedViews

用于缓存滑出屏幕的ViewHodler。 缓存的个数，由RecyclerView#Recycler#mViewCacheMax决定，可以通过RecyclerView#setItemViewCacheSize来设置。

>  
 场景 加入此时屏幕中有10个Item,当用户上滑一个，内容下移一个时，原先第一个的Item中的ViewHodler会被加入到mCachedViews中。 


## 源码解析

首先直接找到在mCachedViews中缓存的位置。

```
       /**
         * internal implementation checks if view is scrapped or attached and throws an exception
         * if so.
         * Public version un-scraps before calling recycle.
         */
        void recycleViewHolderInternal(ViewHolder holder) {<!-- -->
            if (holder.isScrap() || holder.itemView.getParent() != null) {<!-- -->
                throw new IllegalArgumentException(
                        "Scrapped or attached views may not be recycled. isScrap:"
                                + holder.isScrap() + " isAttached:"
                                + (holder.itemView.getParent() != null) + exceptionLabel());
            }

            if (holder.isTmpDetached()) {<!-- -->
                throw new IllegalArgumentException("Tmp detached view should be removed "
                        + "from RecyclerView before it can be recycled: " + holder
                        + exceptionLabel());
            }

            if (holder.shouldIgnore()) {<!-- -->
                throw new IllegalArgumentException("Trying to recycle an ignored view holder. You"
                        + " should first call stopIgnoringView(view) before calling recycle."
                        + exceptionLabel());
            }
            //noinspection unchecked
            final boolean transientStatePreventsRecycling = holder
                    .doesTransientStatePreventRecycling();
            final boolean forceRecycle = mAdapter != null
                    &amp;&amp; transientStatePreventsRecycling
                    &amp;&amp; mAdapter.onFailedToRecycleView(holder);
            boolean cached = false;
            boolean recycled = false;
            if (DEBUG &amp;&amp; mCachedViews.contains(holder)) {<!-- -->
                throw new IllegalArgumentException("cached view received recycle internal? "
                        + holder + exceptionLabel());
            }
            if (forceRecycle || holder.isRecyclable()) {<!-- -->
                if (mViewCacheMax &gt; 0
                        &amp;&amp; !holder.hasAnyOfTheFlags(ViewHolder.FLAG_INVALID
                        | ViewHolder.FLAG_REMOVED
                        | ViewHolder.FLAG_UPDATE
                        | ViewHolder.FLAG_ADAPTER_POSITION_UNKNOWN)) {<!-- -->
                    // Retire oldest cached view
                    int cachedViewSize = mCachedViews.size();
                    if (cachedViewSize &gt;= mViewCacheMax &amp;&amp; cachedViewSize &gt; 0) {<!-- -->
                        recycleCachedViewAt(0);
                        cachedViewSize--;
                    }

                    int targetCacheIndex = cachedViewSize;
                    if (ALLOW_THREAD_GAP_WORK
                            &amp;&amp; cachedViewSize &gt; 0
                            &amp;&amp; !mPrefetchRegistry.lastPrefetchIncludedPosition(holder.mPosition)) {<!-- -->
                        // when adding the view, skip past most recently prefetched views
                        int cacheIndex = cachedViewSize - 1;
                        while (cacheIndex &gt;= 0) {<!-- -->
                            int cachedPos = mCachedViews.get(cacheIndex).mPosition;
                            if (!mPrefetchRegistry.lastPrefetchIncludedPosition(cachedPos)) {<!-- -->
                                break;
                            }
                            cacheIndex--;
                        }
                        targetCacheIndex = cacheIndex + 1;
                    }
                    mCachedViews.add(targetCacheIndex, holder);
                    cached = true;
                }
                if (!cached) {<!-- -->
                    addViewHolderToRecycledViewPool(holder, true);
                    recycled = true;
                }
            } else {<!-- -->
                // NOTE: A view can fail to be recycled when it is scrolled off while an animation
                // runs. In this case, the item is eventually recycled by
                // ItemAnimatorRestoreListener#onAnimationFinished.

                // TODO: consider cancelling an animation when an item is removed scrollBy,
                // to return it to the pool faster
                if (DEBUG) {<!-- -->
                    Log.d(TAG, "trying to recycle a non-recycleable holder. Hopefully, it will "
                            + "re-visit here. We are still removing it from animation lists"
                            + exceptionLabel());
                }
            }
            // even if the holder is not removed, we still call this method so that it is removed
            // from view holder lists.
            mViewInfoStore.removeViewHolder(holder);
            if (!cached &amp;&amp; !recycled &amp;&amp; transientStatePreventsRecycling) {<!-- -->
                holder.mOwnerRecyclerView = null;
            }
        }

```

```
        void recycleCachedViewAt(int cachedViewIndex) {<!-- -->
            if (DEBUG) {<!-- -->
                Log.d(TAG, "Recycling cached view at index " + cachedViewIndex);
            }
            ViewHolder viewHolder = mCachedViews.get(cachedViewIndex);
            if (DEBUG) {<!-- -->
                Log.d(TAG, "CachedViewHolder to be recycled: " + viewHolder);
            }
            addViewHolderToRecycledViewPool(viewHolder, true);
            mCachedViews.remove(cachedViewIndex);
        }


```

此方法逻辑如下：
1. 首先判断ViewHolder是否可以缓存1. 如果mCachedViews的size已经大于等于mViewCacheMax，那么会将超index=0这个位置的ViewHolder放入到回收池中。1. 此时mCachedViews的size不会大于mViewCacheMax，将ViewHodler添加到mCachedViews中
# mViewCacheExtension

直接看源码。

```
    /**
     * ViewCacheExtension is a helper class to provide an additional layer of view caching that can
     * be controlled by the developer.
     * &lt;p&gt;
     * When {@link Recycler#getViewForPosition(int)} is called, Recycler checks attached scrap and
     * first level cache to find a matching View. If it cannot find a suitable View, Recycler will
     * call the {@link #getViewForPositionAndType(Recycler, int, int)} before checking
     * {@link RecycledViewPool}.
     * &lt;p&gt;
     * Note that, Recycler never sends Views to this method to be cached. It is developers
     * responsibility to decide whether they want to keep their Views in this custom cache or let
     * the default recycling policy handle it.
     */
    public abstract static class ViewCacheExtension {<!-- -->

        /**
         * Returns a View that can be binded to the given Adapter position.
         * &lt;p&gt;
         * This method should &lt;b&gt;not&lt;/b&gt; create a new View. Instead, it is expected to return
         * an already created View that can be re-used for the given type and position.
         * If the View is marked as ignored, it should first call
         * {@link LayoutManager#stopIgnoringView(View)} before returning the View.
         * &lt;p&gt;
         * RecyclerView will re-bind the returned View to the position if necessary.
         *
         * @param recycler The Recycler that can be used to bind the View
         * @param position The adapter position
         * @param type     The type of the View, defined by adapter
         * @return A View that is bound to the given position or NULL if there is no View to re-use
         * @see LayoutManager#ignoreView(View)
         */
        @Nullable
        public abstract View getViewForPositionAndType(@NonNull Recycler recycler, int position,
                int type);
    }

```

ViewCacheExtension是提供给开发者自定义View缓存的一个帮助类。 如果使用ViewCacheExtension来自定义缓存，其内容与RecyclerView#Recycler是无关的，需要开发者自己定义View的缓存与回收逻辑。

# RecycledViewPool

RecycledViewPool能够共享多个RecyclerView之间的View。（同一个Adapter） 默认情况下每个RecyclerView会自己创建一个RecycledViewPool，开发者如果需要多个RecycledView复用一个Pool可以通过RecycledView#setRecycledViewPool()来设置。

>  
 场景：当用户上下滑动页面触发回收ViewHolder逻辑，并且mCachedViews中已经存放满了的时候，会放到mRecyclerPool中. 


### 存储结构

```
        private static final int DEFAULT_MAX_SCRAP = 5;
        static class ScrapData {<!-- -->
            final ArrayList&lt;ViewHolder&gt; mScrapHeap = new ArrayList&lt;&gt;();
            int mMaxScrap = DEFAULT_MAX_SCRAP;
            long mCreateRunningAverageNs = 0;
            long mBindRunningAverageNs = 0;
        }
        SparseArray&lt;ScrapData&gt; mScrap = new SparseArray&lt;&gt;();  
        

```
- ViewHolder存储与RecycledViewPool的ScapData数据结构的ArrayList中。- ScrapData则存储在RecycledViewPool中的SparseArray中。- ScrapData中的ArrayList默认的存储上限是5.
### 获取ViewHolder

```
        /**
         * Acquire a ViewHolder of the specified type from the pool, or {@code null} if none are
         * present.
         *
         * @param viewType ViewHolder type.
         * @return ViewHolder of the specified type acquired from the pool, or {@code null} if none
         * are present.
         */
        @Nullable
        public ViewHolder getRecycledView(int viewType) {<!-- -->
            final ScrapData scrapData = mScrap.get(viewType);
            if (scrapData != null &amp;&amp; !scrapData.mScrapHeap.isEmpty()) {<!-- -->
                final ArrayList&lt;ViewHolder&gt; scrapHeap = scrapData.mScrapHeap;
                return scrapHeap.remove(scrapHeap.size() - 1);
            }
            return null;
        }

```
1. 根据viewType去SparseArray中获取到ScrapData。1. 从ScrapData的ArrayList中返回一个ViewHodler出去，并且从该mScrapHeap中移除。
### 回收ViewHolder

```
        /**
         * Add a scrap ViewHolder to the pool.
         * &lt;p&gt;
         * If the pool is already full for that ViewHolder's type, it will be immediately discarded.
         *
         * @param scrap ViewHolder to be added to the pool.
         */
        public void putRecycledView(ViewHolder scrap) {<!-- -->
            final int viewType = scrap.getItemViewType();
            final ArrayList&lt;ViewHolder&gt; scrapHeap = getScrapDataForType(viewType).mScrapHeap;
            if (mScrap.get(viewType).mMaxScrap &lt;= scrapHeap.size()) {<!-- -->
                return;
            }
            if (DEBUG &amp;&amp; scrapHeap.contains(scrap)) {<!-- -->
                throw new IllegalArgumentException("this scrap item already exists");
            }
            scrap.resetInternal();
            scrapHeap.add(scrap);
        }

```

```
        void resetInternal() {<!-- -->
            mFlags = 0;
            mPosition = NO_POSITION;
            mOldPosition = NO_POSITION;
            mItemId = NO_ID;
            mPreLayoutPosition = NO_POSITION;
            mIsRecyclableCount = 0;
            mShadowedHolder = null;
            mShadowingHolder = null;
            clearPayload();
            mWasImportantForAccessibilityBeforeHidden = ViewCompat.IMPORTANT_FOR_ACCESSIBILITY_AUTO;
            mPendingAccessibilityState = PENDING_ACCESSIBILITY_STATE_NOT_SET;
            clearNestedRecyclerViewIfNotNested(this);
        }

```
1. 先获取到ViewHolder的viewType，然后根据viewType从SparseArray中获取到ScrapData1. 如果ScrapData中的ArrayList已经到达上限，那么久直接返回，不回收。1. 如果ScrapData中的ArrayList已经包含了该ViewHolder，那么直接报错。1. 在ViewHolder放入到回收池之前清除数据。1. 将该ViewHolder放入到ScrapData的ArrayList中。
# 总结

RecyclerView的回收机制中主要是以下几个模块：

|模块|描述|场景
|------
|mAttachedScrap|用于存储ViewHolder已经从RecyclerView上移除，但是仍有可能被复用的View。|通过adapter设置notifyItemChanged（index），当前屏幕中index以外的其他item都会被存储到mAttachedScrap中。
|mChangedScrap|用于存储ViewHolder仍在RecyclerView上，但是数据已经过时，需要被更新的View。|通过adapter设置notifyItemChanged（index），如果当前index显示在屏幕中，这个index的ViewHodler会被存储到mChangedScrap中。
|mCachedViews|用于缓存滑出屏幕的ViewHodler。当mCachedViews满了之后，会把最老的ViewHolder的放入到RecyclerViewPool。|加入此时屏幕中有10个Item,当用户上滑一个，内容下移一个时，原先第一个的Item中的ViewHodler会被加入到mCachedViews中。
|mViewCacheExtension|ViewCacheExtension是提供给开发者自定义View缓存的一个帮助类。如果使用ViewCacheExtension来自定义缓存，需要开发者自己定义View的缓存与回收逻辑。|不常用，开发者可自由定义。
|mRecyclerPool|RecycledViewPool能够共享多个RecyclerView之间的View。（同一个Adapter）默认情况下每个RecyclerView会自己创建一个RecycledViewPool，开发者如果需要多个RecycledView复用一个Pool可以通过RecycledView#setRecycledViewPool()来设置。|屏幕上下滑动缓存ViewHolder，且mCachedViews中放满的情况下会放到RecycledViewPool中

>  
 注： 
 - 与当前的RecyclerView一一对应： mAttachedScrap、mChangedScrap、mCachedViews- 可能多个RecyclerView共享： mViewCacheExtension、mRecyclerPool 

