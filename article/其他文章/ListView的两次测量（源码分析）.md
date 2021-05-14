#ListView的两次测量（源码分析）
# 前言

ListView是Android开发者最常见的控件之一，但是真的很少有人会去思考他是如何实现的，包括笔者也是。 最近有学长正好问到这个问题，笔者当场懵逼。 于是痛定思痛，决定阅读其源码，了解一下**ListView的测量**原理。一方面是提高自己阅读源码的自学能力，另一方面是打算让自己对View的测量的理解更进一步。

# 进入正题

在此，不得不提一个概念：

>  
 任何一个View，在展示到界面上之前都会经历至少两次onMeasure()和两次onLayout()的过程。 


这在平时其实对我们没有什么影响，但是对于ListView这种复杂的控件来讲就不一样了。经过ListView源码中诸多的if-else语句的过滤，两次执行的代码看上去是完全不一样的。 所以我们源码的分析也分成了第一次测量与第二次测量。

为了方便读者的理解，我们附上官网ListView的继承结构： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039331610.png" alt="这里写图片描述">

## 寻找入口

ListView是一个非常复杂的控件，仅仅我们经常用的功能就包括：复用回收、设置HeadView、设置FootView、设置Adapter、设置分割线、设置当前位置等等。 如果要完全把ListView进行分析，那需要花费大量的时间和文笔。我们首先要清晰自己分析需求，笔者此文也是仅针对**ListView的测量**进行分析。 我相信大家首先想到分析的方法就是setAdapter()，因为只有当设置adapter之后，ListView才会拥有子View并进行显示，但是如上所说，ListView是一个非常复杂的控件，通过对setAdapter()分析后，很容易可以发现其中只是获取到了该adapter，具体绘制内容并不在里面，如下所示：

```
public void setAdapter(ListAdapter adapter) {
        if (mAdapter != null &amp;&amp; mDataSetObserver != null) {
            mAdapter.unregisterDataSetObserver(mDataSetObserver);
        }

        resetList();
        mRecycler.clear();

        if (mHeaderViewInfos.size() &gt; 0|| mFooterViewInfos.size() &gt; 0) {
            mAdapter = new HeaderViewListAdapter(mHeaderViewInfos, mFooterViewInfos, adapter);
        } else {
            mAdapter = adapter;
        }

        mOldSelectedPosition = INVALID_POSITION;
        mOldSelectedRowId = INVALID_ROW_ID;

        // AbsListView#setAdapter will update choice mode states.
        super.setAdapter(adapter);

        if (mAdapter != null) {
            mAreAllItemsSelectable = mAdapter.areAllItemsEnabled();
            mOldItemCount = mItemCount;
            mItemCount = mAdapter.getCount();
            checkFocus();

            mDataSetObserver = new AdapterDataSetObserver();
            mAdapter.registerDataSetObserver(mDataSetObserver);

            mRecycler.setViewTypeCount(mAdapter.getViewTypeCount());

            int position;
            if (mStackFromBottom) {
                position = lookForSelectablePosition(mItemCount - 1, false);
            } else {
                position = lookForSelectablePosition(0, true);
            }
            setSelectedPositionInt(position);
            setNextSelectedPositionInt(position);

            if (mItemCount == 0) {
                // Nothing selected
                checkSelectionChanged();
            }
        } else {
            mAreAllItemsSelectable = true;
            checkFocus();
            // Nothing selected
            checkSelectionChanged();
        }

        requestLayout();
    }

```

那么我们只能寻找最常规的方法了。View的测量是依靠onMeasure()以及onLayout()方法。 我们使用ListView一般就是占用整个屏幕，onMeasure()没有特别需要分析的必要。 我们主要就讲一下onLayout()。

# ListView第一次测量

首先我们可以发现在ListView中并不存在onLayout()这个方法。 那么这个方法就一定是写在ListView的父类AbsListView中了。 我们可以找到如下：

```
protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);

        mInLayout = true;

        final int childCount = getChildCount();
        if (changed) {
            for (int i = 0; i &lt; childCount; i++) {
                getChildAt(i).forceLayout();
            }
            mRecycler.markChildrenDirty();
        }

        layoutChildren();
        mInLayout = false;

        mOverscrollMax = (b - t) / OVERSCROLL_LIMIT_DIVISOR;

        // TODO: Move somewhere sane. This doesn't belong in onLayout().
        if (mFastScroll != null) {
            mFastScroll.onItemCountChanged(getChildCount(), mItemCount);
        }
    }

```

我们可以看到，onLayout()方法中并没有做什么复杂的逻辑操作，主要就是一个判断，如果ListView的大小或者位置发生了变化，ListView所有的子布局都强制进行重绘。layoutChildren()这个方法，从方法名上我们就可以猜出这个方法是用来进行子元素布局的。 但是我们点开后如下所示：

```
    /**
     * Subclasses must override this method to layout their children.
     */
    protected void layoutChildren() {
    }

```

我们发现这是一个空方法。其实很容易理解，ListView和GridView都继承自AbsListView，子部局的排版方式当然是继承后写到子类中了。 然后我们跳转到ListView的layoutChildren方法：

```
protected void layoutChildren() {
        final boolean blockLayoutRequests = mBlockLayoutRequests;
        if (blockLayoutRequests) {
            return;
        }

        mBlockLayoutRequests = true;

        try {
            super.layoutChildren();

            invalidate();

            if (mAdapter == null) {
                resetList();
                invokeOnItemScrollListener();
                return;
            }

            final int childrenTop = mListPadding.top;
            final int childrenBottom = mBottom - mTop - mListPadding.bottom;
            final int childCount = getChildCount();

            int index = 0;
            int delta = 0;

            View sel;
            View oldSel = null;
            View oldFirst = null;
            View newSel = null;

            // Remember stuff we will need down below
            switch (mLayoutMode) {
            case LAYOUT_SET_SELECTION:
                index = mNextSelectedPosition - mFirstPosition;
                if (index &gt;= 0 &amp;&amp; index &lt; childCount) {
                    newSel = getChildAt(index);
                }
                break;
            case LAYOUT_FORCE_TOP:
            case LAYOUT_FORCE_BOTTOM:
            case LAYOUT_SPECIFIC:
            case LAYOUT_SYNC:
                break;
            case LAYOUT_MOVE_SELECTION:
            default:
                // Remember the previously selected view
                index = mSelectedPosition - mFirstPosition;
                if (index &gt;= 0 &amp;&amp; index &lt; childCount) {
                    oldSel = getChildAt(index);
                }

                // Remember the previous first child
                oldFirst = getChildAt(0);

                if (mNextSelectedPosition &gt;= 0) {
                    delta = mNextSelectedPosition - mSelectedPosition;
                }

                // Caution: newSel might be null
                newSel = getChildAt(index + delta);
            }


            boolean dataChanged = mDataChanged;
            if (dataChanged) {
                handleDataChanged();
            }

            // Handle the empty set by removing all views that are visible
            // and calling it a day
            if (mItemCount == 0) {
                resetList();
                invokeOnItemScrollListener();
                return;
            } else if (mItemCount != mAdapter.getCount()) {
                throw new IllegalStateException("The content of the adapter has changed but "
                        + "ListView did not receive a notification. Make sure the content of "
                        + "your adapter is not modified from a background thread, but only from "
                        + "the UI thread. Make sure your adapter calls notifyDataSetChanged() "
                        + "when its content changes. [in ListView(" + getId() + ", " + getClass()
                        + ") with Adapter(" + mAdapter.getClass() + ")]");
            }

            setSelectedPositionInt(mNextSelectedPosition);

            AccessibilityNodeInfo accessibilityFocusLayoutRestoreNode = null;
            View accessibilityFocusLayoutRestoreView = null;
            int accessibilityFocusPosition = INVALID_POSITION;

            // Remember which child, if any, had accessibility focus. This must
            // occur before recycling any views, since that will clear
            // accessibility focus.
            final ViewRootImpl viewRootImpl = getViewRootImpl();
            if (viewRootImpl != null) {
                final View focusHost = viewRootImpl.getAccessibilityFocusedHost();
                if (focusHost != null) {
                    final View focusChild = getAccessibilityFocusedChild(focusHost);
                    if (focusChild != null) {
                        if (!dataChanged || isDirectChildHeaderOrFooter(focusChild)
                                || focusChild.hasTransientState() || mAdapterHasStableIds) {
                            // The views won't be changing, so try to maintain
                            // focus on the current host and virtual view.
                            accessibilityFocusLayoutRestoreView = focusHost;
                            accessibilityFocusLayoutRestoreNode = viewRootImpl
                                    .getAccessibilityFocusedVirtualView();
                        }

                        // If all else fails, maintain focus at the same
                        // position.
                        accessibilityFocusPosition = getPositionForView(focusChild);
                    }
                }
            }

            View focusLayoutRestoreDirectChild = null;
            View focusLayoutRestoreView = null;

            // Take focus back to us temporarily to avoid the eventual call to
            // clear focus when removing the focused child below from messing
            // things up when ViewAncestor assigns focus back to someone else.
            final View focusedChild = getFocusedChild();
            if (focusedChild != null) {
                // TODO: in some cases focusedChild.getParent() == null

                // We can remember the focused view to restore after re-layout
                // if the data hasn't changed, or if the focused position is a
                // header or footer.
                if (!dataChanged || isDirectChildHeaderOrFooter(focusedChild)
                        || focusedChild.hasTransientState() || mAdapterHasStableIds) {
                    focusLayoutRestoreDirectChild = focusedChild;
                    // Remember the specific view that had focus.
                    focusLayoutRestoreView = findFocus();
                    if (focusLayoutRestoreView != null) {
                        // Tell it we are going to mess with it.
                        focusLayoutRestoreView.onStartTemporaryDetach();
                    }
                }
                requestFocus();
            }

            // Pull all children into the RecycleBin.
            // These views will be reused if possible
            final int firstPosition = mFirstPosition;
            final RecycleBin recycleBin = mRecycler;
            if (dataChanged) {
                for (int i = 0; i &lt; childCount; i++) {
                    recycleBin.addScrapView(getChildAt(i), firstPosition+i);
                }
            } else {
                recycleBin.fillActiveViews(childCount, firstPosition);
            }

            // Clear out old views
            detachAllViewsFromParent();
            recycleBin.removeSkippedScrap();

            switch (mLayoutMode) {
            case LAYOUT_SET_SELECTION:
                if (newSel != null) {
                    sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
                } else {
                    sel = fillFromMiddle(childrenTop, childrenBottom);
                }
                break;
            case LAYOUT_SYNC:
                sel = fillSpecific(mSyncPosition, mSpecificTop);
                break;
            case LAYOUT_FORCE_BOTTOM:
                sel = fillUp(mItemCount - 1, childrenBottom);
                adjustViewsUpOrDown();
                break;
            case LAYOUT_FORCE_TOP:
                mFirstPosition = 0;
                sel = fillFromTop(childrenTop);
                adjustViewsUpOrDown();
                break;
            case LAYOUT_SPECIFIC:
                sel = fillSpecific(reconcileSelectedPosition(), mSpecificTop);
                break;
            case LAYOUT_MOVE_SELECTION:
                sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
                break;
            default:
                if (childCount == 0) {
                    if (!mStackFromBottom) {
                        final int position = lookForSelectablePosition(0, true);
                        setSelectedPositionInt(position);
                        sel = fillFromTop(childrenTop);
                    } else {
                        final int position = lookForSelectablePosition(mItemCount - 1, false);
                        setSelectedPositionInt(position);
                        sel = fillUp(mItemCount - 1, childrenBottom);
                    }
                } else {
                    if (mSelectedPosition &gt;= 0 &amp;&amp; mSelectedPosition &lt; mItemCount) {
                        sel = fillSpecific(mSelectedPosition,
                                oldSel == null ? childrenTop : oldSel.getTop());
                    } else if (mFirstPosition &lt; mItemCount) {
                        sel = fillSpecific(mFirstPosition,
                                oldFirst == null ? childrenTop : oldFirst.getTop());
                    } else {
                        sel = fillSpecific(0, childrenTop);
                    }
                }
                break;
            }

            // Flush any cached views that did not get reused above
            recycleBin.scrapActiveViews();

            if (sel != null) {
                // The current selected item should get focus if items are
                // focusable.
                if (mItemsCanFocus &amp;&amp; hasFocus() &amp;&amp; !sel.hasFocus()) {
                    final boolean focusWasTaken = (sel == focusLayoutRestoreDirectChild &amp;&amp;
                            focusLayoutRestoreView != null &amp;&amp;
                            focusLayoutRestoreView.requestFocus()) || sel.requestFocus();
                    if (!focusWasTaken) {
                        // Selected item didn't take focus, but we still want to
                        // make sure something else outside of the selected view
                        // has focus.
                        final View focused = getFocusedChild();
                        if (focused != null) {
                            focused.clearFocus();
                        }
                        positionSelector(INVALID_POSITION, sel);
                    } else {
                        sel.setSelected(false);
                        mSelectorRect.setEmpty();
                    }
                } else {
                    positionSelector(INVALID_POSITION, sel);
                }
                mSelectedTop = sel.getTop();
            } else {
                final boolean inTouchMode = mTouchMode == TOUCH_MODE_TAP
                        || mTouchMode == TOUCH_MODE_DONE_WAITING;
                if (inTouchMode) {
                    // If the user's finger is down, select the motion position.
                    final View child = getChildAt(mMotionPosition - mFirstPosition);
                    if (child != null) {
                        positionSelector(mMotionPosition, child);
                    }
                } else if (mSelectorPosition != INVALID_POSITION) {
                    // If we had previously positioned the selector somewhere,
                    // put it back there. It might not match up with the data,
                    // but it's transitioning out so it's not a big deal.
                    final View child = getChildAt(mSelectorPosition - mFirstPosition);
                    if (child != null) {
                        positionSelector(mSelectorPosition, child);
                    }
                } else {
                    // Otherwise, clear selection.
                    mSelectedTop = 0;
                    mSelectorRect.setEmpty();
                }

                // Even if there is not selected position, we may need to
                // restore focus (i.e. something focusable in touch mode).
                if (hasFocus() &amp;&amp; focusLayoutRestoreView != null) {
                    focusLayoutRestoreView.requestFocus();
                }
            }

            // Attempt to restore accessibility focus, if necessary.
            if (viewRootImpl != null) {
                final View newAccessibilityFocusedView = viewRootImpl.getAccessibilityFocusedHost();
                if (newAccessibilityFocusedView == null) {
                    if (accessibilityFocusLayoutRestoreView != null
                            &amp;&amp; accessibilityFocusLayoutRestoreView.isAttachedToWindow()) {
                        final AccessibilityNodeProvider provider =
                                accessibilityFocusLayoutRestoreView.getAccessibilityNodeProvider();
                        if (accessibilityFocusLayoutRestoreNode != null &amp;&amp; provider != null) {
                            final int virtualViewId = AccessibilityNodeInfo.getVirtualDescendantId(
                                    accessibilityFocusLayoutRestoreNode.getSourceNodeId());
                            provider.performAction(virtualViewId,
                                    AccessibilityNodeInfo.ACTION_ACCESSIBILITY_FOCUS, null);
                        } else {
                            accessibilityFocusLayoutRestoreView.requestAccessibilityFocus();
                        }
                    } else if (accessibilityFocusPosition != INVALID_POSITION) {
                        // Bound the position within the visible children.
                        final int position = MathUtils.constrain(
                                accessibilityFocusPosition - mFirstPosition, 0,
                                getChildCount() - 1);
                        final View restoreView = getChildAt(position);
                        if (restoreView != null) {
                            restoreView.requestAccessibilityFocus();
                        }
                    }
                }
            }

            // Tell focus view we are done mucking with it, if it is still in
            // our view hierarchy.
            if (focusLayoutRestoreView != null
                    &amp;&amp; focusLayoutRestoreView.getWindowToken() != null) {
                focusLayoutRestoreView.onFinishTemporaryDetach();
            }
            
            mLayoutMode = LAYOUT_NORMAL;
            mDataChanged = false;
            if (mPositionScrollAfterLayout != null) {
                post(mPositionScrollAfterLayout);
                mPositionScrollAfterLayout = null;
            }
            mNeedSync = false;
            setNextSelectedPositionInt(mSelectedPosition);

            updateScrollIndicators();

            if (mItemCount &gt; 0) {
                checkSelectionChanged();
            }

            invokeOnItemScrollListener();
        } finally {
            if (!blockLayoutRequests) {
                mBlockLayoutRequests = false;
            }
        }
    }

```

代码有三百多行，说实话笔者刚开始看的时候是非常懵逼的。 但是我们可以根据函数名进行逻辑的筛选： 1、mAdapter为空等为特殊情况，我们不需要考虑，我们想得到的是正常情况下ListView的测量方法。 2、我们可以看到有很多参数都与“focus”相关，我们仅仅想知道ListView的测量方法，至于ListView其他的功能是怎么实现的，我们暂时就放一边了。 经过筛选后，我们可以得到以下代码：

```
// Pull all children into the RecycleBin.
            // These views will be reused if possible
            final int firstPosition = mFirstPosition;
            final RecycleBin recycleBin = mRecycler;
            if (dataChanged) {
                for (int i = 0; i &lt; childCount; i++) {
                    recycleBin.addScrapView(getChildAt(i), firstPosition+i);
                }
            } else {
                recycleBin.fillActiveViews(childCount, firstPosition);
            }

            // Clear out old views
            detachAllViewsFromParent();
            recycleBin.removeSkippedScrap();

            switch (mLayoutMode) {
            case LAYOUT_SET_SELECTION:
                if (newSel != null) {
                    sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
                } else {
                    sel = fillFromMiddle(childrenTop, childrenBottom);
                }
                break;
            case LAYOUT_SYNC:
                sel = fillSpecific(mSyncPosition, mSpecificTop);
                break;
            case LAYOUT_FORCE_BOTTOM:
                sel = fillUp(mItemCount - 1, childrenBottom);
                adjustViewsUpOrDown();
                break;
            case LAYOUT_FORCE_TOP:
                mFirstPosition = 0;
                sel = fillFromTop(childrenTop);
                adjustViewsUpOrDown();
                break;
            case LAYOUT_SPECIFIC:
                sel = fillSpecific(reconcileSelectedPosition(), mSpecificTop);
                break;
            case LAYOUT_MOVE_SELECTION:
                sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
                break;
            default:
                if (childCount == 0) {
                    if (!mStackFromBottom) {
                        final int position = lookForSelectablePosition(0, true);
                        setSelectedPositionInt(position);
                        sel = fillFromTop(childrenTop);
                    } else {
                        final int position = lookForSelectablePosition(mItemCount - 1, false);
                        setSelectedPositionInt(position);
                        sel = fillUp(mItemCount - 1, childrenBottom);
                    }
                } else {
                    if (mSelectedPosition &gt;= 0 &amp;&amp; mSelectedPosition &lt; mItemCount) {
                        sel = fillSpecific(mSelectedPosition,
                                oldSel == null ? childrenTop : oldSel.getTop());
                    } else if (mFirstPosition &lt; mItemCount) {
                        sel = fillSpecific(mFirstPosition,
                                oldFirst == null ? childrenTop : oldFirst.getTop());
                    } else {
                        sel = fillSpecific(0, childrenTop);
                    }
                }
                break;
            }

            // Flush any cached views that did not get reused above
            recycleBin.scrapActiveViews();

```

首先我们可以看switch (mLayoutMode){}之前的逻辑，根据注释可得，这些是ListView的复用逻辑，可以排除。

```
// Pull all children into the RecycleBin.
// These views will be reused if possible

//Clear out old views

```

那么我们就需要看switch (mLayoutMode){}中的逻辑了，首先我们要知道会进入mLayoutMode的什么模式。 我们可以发现ListView是不带有mLayoutMode这个参数的。那么，我们直接进入AbsListView进行查看。

```
/**
     * Controls how the next layout will happen
     */
    int mLayoutMode = LAYOUT_NORMAL;

```

我们可以发现mLayoutMode 默认情况下是LAYOUT_NORMAL模式，在switch中不存在，那么便会进入default中。 接下来我们查看default的代码：

```
if (childCount == 0) {
                    if (!mStackFromBottom) {
                        final int position = lookForSelectablePosition(0, true);
                        setSelectedPositionInt(position);
                        sel = fillFromTop(childrenTop);
                    } else {
                        final int position = lookForSelectablePosition(mItemCount - 1, false);
                        setSelectedPositionInt(position);
                        sel = fillUp(mItemCount - 1, childrenBottom);
                    }
                } else {
                    if (mSelectedPosition &gt;= 0 &amp;&amp; mSelectedPosition &lt; mItemCount) {
                        sel = fillSpecific(mSelectedPosition,
                                oldSel == null ? childrenTop : oldSel.getTop());
                    } else if (mFirstPosition &lt; mItemCount) {
                        sel = fillSpecific(mFirstPosition,
                                oldFirst == null ? childrenTop : oldFirst.getTop());
                    } else {
                        sel = fillSpecific(0, childrenTop);
                    }
                }

```

首先我们先思考一下我们一般使用LIstView的情况，我们一般会下xml中如下定义：

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    &gt;

    &lt;ListView
        android:id="@+id/lv_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent"&gt;
        
    &lt;/ListView&gt;
&lt;/RelativeLayout&gt;

```

我们可以看到，ListView刚开始是没有子部局的。也是说childCount =0； （只有在使用SetAdapter()之后，ListView才会有子部局，当然在xml中也不会体现出来） 那么，我们接下来就很方便了。childCount =0，并且我们默认的布局都是从上往下的，因此我们就会跳入fillFromTop()这个方法。

```
/**
     * Fills the list from top to bottom, starting with mFirstPosition
     *
     * @param nextTop The location where the top of the first item should be
     *        drawn
     *
     * @return The view that is currently selected
     */
private View fillFromTop(int nextTop) {
        mFirstPosition = Math.min(mFirstPosition, mSelectedPosition);
        mFirstPosition = Math.min(mFirstPosition, mItemCount - 1);
        if (mFirstPosition &lt; 0) {
            mFirstPosition = 0;
        }
        return fillDown(mFirstPosition, nextTop);
    }

```

从这个方法的注释中可以看出，它所负责的主要任务就是从mFirstPosition开始，自顶至底去填充ListView。但是这个方法本身并没有什么逻辑，因此我们可以确定逻辑在fillDown()这个函数中：

```
/**
     * Fills the list from pos down to the end of the list view.
     *
     * @param pos The first position to put in the list
     *
     * @param nextTop The location where the top of the item associated with pos
     *        should be drawn
     *
     * @return The view that is currently selected, if it happens to be in the
     *         range that we draw.
     */
private View fillDown(int pos, int nextTop) {
        View selectedView = null;

        int end = (mBottom - mTop);
        if ((mGroupFlags &amp; CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
            end -= mListPadding.bottom;
        }

        while (nextTop &lt; end &amp;&amp; pos &lt; mItemCount) {
            // is this the selected item?
            boolean selected = pos == mSelectedPosition;
            View child = makeAndAddView(pos, nextTop, true, mListPadding.left, selected);

            nextTop = child.getBottom() + mDividerHeight;
            if (selected) {
                selectedView = child;
            }
            pos++;
        }

        setVisibleRangeHint(mFirstPosition, mFirstPosition + getChildCount() - 1);
        return selectedView;
    }

```

这时候我们看到了一个获取到child这个View的方法makeAndAddView()，于是再进方法内查看：

```
/**
     * Obtain the view and add it to our list of children. The view can be made
     * fresh, converted from an unused view, or used as is if it was in the
     * recycle bin.
     *
     * @param position Logical position in the list
     * @param y Top or bottom edge of the view to add
     * @param flow If flow is true, align top edge to y. If false, align bottom
     *        edge to y.
     * @param childrenLeft Left edge where children should be positioned
     * @param selected Is this position selected?
     * @return View that was added
     */
    private View makeAndAddView(int position, int y, boolean flow, int childrenLeft,
            boolean selected) {
        View child;


        if (!mDataChanged) {
            // Try to use an existing view for this position
            child = mRecycler.getActiveView(position);
            if (child != null) {
                // Found it -- we're using an existing child
                // This just needs to be positioned
                setupChild(child, position, y, flow, childrenLeft, selected, true);

                return child;
            }
        }

        // Make a new view for this position, or convert an unused view if possible
        child = obtainView(position, mIsScrap);

        // This needs to be positioned and measured
        setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);

        return child;
    }

```

这时候我们看注释：

>  
 获取到View并且把它加入到list的子群，View可以被刷新。 


那么我们可以推得，这个把View加入到list中的函数一定是ListView的测量与布局的函数了，即setupChild():

```
/**
     * Add a view as a child and make sure it is measured (if necessary) and
     * positioned properly.
     *
     * @param child The view to add
     * @param position The position of this child
     * @param y The y position relative to which this view will be positioned
     * @param flowDown If true, align top edge to y. If false, align bottom
     *        edge to y.
     * @param childrenLeft Left edge where children should be positioned
     * @param selected Is this position selected?
     * @param recycled Has this view been pulled from the recycle bin? If so it
     *        does not need to be remeasured.
     */
    private void setupChild(View child, int position, int y, boolean flowDown, int childrenLeft,
            boolean selected, boolean recycled) {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "setupListItem");

        final boolean isSelected = selected &amp;&amp; shouldShowSelector();
        final boolean updateChildSelected = isSelected != child.isSelected();
        final int mode = mTouchMode;
        final boolean isPressed = mode &gt; TOUCH_MODE_DOWN &amp;&amp; mode &lt; TOUCH_MODE_SCROLL &amp;&amp;
                mMotionPosition == position;
        final boolean updateChildPressed = isPressed != child.isPressed();
        final boolean needToMeasure = !recycled || updateChildSelected || child.isLayoutRequested();

        // Respect layout params that are already in the view. Otherwise make some up...
        // noinspection unchecked
        AbsListView.LayoutParams p = (AbsListView.LayoutParams) child.getLayoutParams();
        if (p == null) {
            p = (AbsListView.LayoutParams) generateDefaultLayoutParams();
        }
        p.viewType = mAdapter.getItemViewType(position);

        if ((recycled &amp;&amp; !p.forceAdd) || (p.recycledHeaderFooter
                &amp;&amp; p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER)) {
            attachViewToParent(child, flowDown ? -1 : 0, p);
        } else {
            p.forceAdd = false;
            if (p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
                p.recycledHeaderFooter = true;
            }
            addViewInLayout(child, flowDown ? -1 : 0, p, true);
        }

        if (updateChildSelected) {
            child.setSelected(isSelected);
        }

        if (updateChildPressed) {
            child.setPressed(isPressed);
        }

        if (mChoiceMode != CHOICE_MODE_NONE &amp;&amp; mCheckStates != null) {
            if (child instanceof Checkable) {
                ((Checkable) child).setChecked(mCheckStates.get(position));
            } else if (getContext().getApplicationInfo().targetSdkVersion
                    &gt;= android.os.Build.VERSION_CODES.HONEYCOMB) {
                child.setActivated(mCheckStates.get(position));
            }
        }

        if (needToMeasure) {
            final int childWidthSpec = ViewGroup.getChildMeasureSpec(mWidthMeasureSpec,
                    mListPadding.left + mListPadding.right, p.width);
            final int lpHeight = p.height;
            final int childHeightSpec;
            if (lpHeight &gt; 0) {
                childHeightSpec = MeasureSpec.makeMeasureSpec(lpHeight, MeasureSpec.EXACTLY);
            } else {
                childHeightSpec = MeasureSpec.makeSafeMeasureSpec(getMeasuredHeight(),
                        MeasureSpec.UNSPECIFIED);
            }
            child.measure(childWidthSpec, childHeightSpec);
        } else {
            cleanupLayoutState(child);
        }

        final int w = child.getMeasuredWidth();
        final int h = child.getMeasuredHeight();
        final int childTop = flowDown ? y : y - h;

        if (needToMeasure) {
            final int childRight = childrenLeft + w;
            final int childBottom = childTop + h;
            child.layout(childrenLeft, childTop, childRight, childBottom);
        } else {
            child.offsetLeftAndRight(childrenLeft - child.getLeft());
            child.offsetTopAndBottom(childTop - child.getTop());
        }

        if (mCachingStarted &amp;&amp; !child.isDrawingCacheEnabled()) {
            child.setDrawingCacheEnabled(true);
        }

        if (recycled &amp;&amp; (((AbsListView.LayoutParams)child.getLayoutParams()).scrappedFromPosition)
                != position) {
            child.jumpDrawablesToCurrentState();
        }

        Trace.traceEnd(Trace.TRACE_TAG_VIEW);
    }

```

到这里，对于自定义过ViewGroup的onLayout()方法的读者来讲就会非常熟悉了。 由于此篇文章仅仅对于ListView测量以及布局的研究，此处对于needToMeasure这个boolean条件的判断就不讨论了，我们直接看needToMeasure为true时会进行的测量，首先是child的measure：

```
AbsListView.LayoutParams p = (AbsListView.LayoutParams) child.getLayoutParams();

```

```
if (needToMeasure) {
            final int childWidthSpec = ViewGroup.getChildMeasureSpec(mWidthMeasureSpec,
                    mListPadding.left + mListPadding.right, p.width);
            final int lpHeight = p.height;
            final int childHeightSpec;
            if (lpHeight &gt; 0) {
                childHeightSpec = MeasureSpec.makeMeasureSpec(lpHeight, MeasureSpec.EXACTLY);
            } else {
                childHeightSpec = MeasureSpec.makeSafeMeasureSpec(getMeasuredHeight(),
                        MeasureSpec.UNSPECIFIED);
            }

```

首先获取到子View宽度的MeasureSpec。padding为ListView的左右padding之和，宽度即为child的宽度。 然后获取到高度的MeasureSpec，高度即为child的高度。 接着是最关键的子View的layout，函数如下：

```
        final int w = child.getMeasuredWidth();
        final int h = child.getMeasuredHeight();
        final int childTop = flowDown ? y : y - h;

        if (needToMeasure) {
            final int childRight = childrenLeft + w;
            final int childBottom = childTop + h;
            child.layout(childrenLeft, childTop, childRight, childBottom);
        } else {
            child.offsetLeftAndRight(childrenLeft - child.getLeft());
            child.offsetTopAndBottom(childTop - child.getTop());
        }

```

我们可以看到childRight 就是等于childrenLeft 加上子View的宽度，而childBottom就是等于childTop 加上子View的高度。 可知其实layout的关键是在childrenLeft 和childTop 上。

childrenLeft 是由setupChild()的参数之一。 而childTop 得到的方式如下：

```
final int childTop = flowDown ? y : y - h;

```

这时候我们再看setupChild（）这个函数：

```
private void setupChild(View child, int position, int y, boolean flowDown, int childrenLeft,
            boolean selected, boolean recycled)

```

我们可以发现childrenLeft 和childTop 都是取决于传递过来的参数，然后我们回到makeAndAddView()这个方法中，makeAndAddView这个方法如下：

```
makeAndAddView(int position, int y, boolean flow, int childrenLeft,
            boolean selected)

```

setupChild（）在makeAndAddView（）中的调用如下：

```
if (child != null) {
                // Found it -- we're using an existing child
                // This just needs to be positioned
                setupChild(child, position, y, flow, childrenLeft, selected, true);

                return child;
            }

```

我们发现这些参数还是被传递过来的，所以我们还需要回去找： 这时候就是回到fillDown中了：

```
while (nextTop &lt; end &amp;&amp; pos &lt; mItemCount) {
            // is this the selected item?
            boolean selected = pos == mSelectedPosition;
            View child = makeAndAddView(pos, nextTop, true, mListPadding.left, selected);

            nextTop = child.getBottom() + mDividerHeight;
            if (selected) {
                selectedView = child;
            }
            pos++;
        }

```

终于我们找到了参数传递的源头！ 于是我们再看layout函数：

```
final int w = child.getMeasuredWidth();
        final int h = child.getMeasuredHeight();
        final int childTop = flowDown ? y : y - h;

        if (needToMeasure) {
            final int childRight = childrenLeft + w;
            final int childBottom = childTop + h;
            child.layout(childrenLeft, childTop, childRight, childBottom);
        } else {
            child.offsetLeftAndRight(childrenLeft - child.getLeft());
            child.offsetTopAndBottom(childTop - child.getTop());
        }

```

childrenLeft为ListView的PaddingLeft。 而决定childTop 的为flowDown 这个boolean和y这个int。我们可以看到，flowDown 在这里的值为true，而y则是由nextTop这个值传回来的，我们需要再回去找，接着就到了fillDown和fillFromTop这两个函数，最终回到了ListView中的layoutChild：

```
private View fillDown(int pos, int nextTop) {
        View selectedView = null;

        int end = (mBottom - mTop);
        if ((mGroupFlags &amp; CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
            end -= mListPadding.bottom;
        }

        while (nextTop &lt; end &amp;&amp; pos &lt; mItemCount) {
            // is this the selected item?
            boolean selected = pos == mSelectedPosition;
            View child = makeAndAddView(pos, nextTop, true, mListPadding.left, selected);

            nextTop = child.getBottom() + mDividerHeight;
            if (selected) {
                selectedView = child;
            }
            pos++;
        }

        setVisibleRangeHint(mFirstPosition, mFirstPosition + getChildCount() - 1);
        return selectedView;
    }

```

```
View fillFromTop(int nextTop)

```

```
protected void layoutChildren() {
…………………………………………
 final int childrenTop = mListPadding.top;
            final int childrenBottom = mBottom - mTop - mListPadding.bottom;
            final int childCount = getChildCount();
…………………………………………
if (childCount == 0) {
                    if (!mStackFromBottom) {
                        final int position = lookForSelectablePosition(0, true);
                        setSelectedPositionInt(position);
                        sel = fillFromTop(childrenTop);
                    }
…………………………………………
}

```

然后我们可以在layoutChildren这个函数中发现，最初的nextTop这个值为ListView的PaddingTop。 而接下来是在fillDown这个函数中累加，如下：

```
nextTop = child.getBottom() + mDividerHeight;

```

nextTop 等于上一个子View的底部的位置加上分隔线的高度。 终于layout的所有参数我们都看懂了！

# 第一次测量结果

我们再看layout的函数

```
final int w = child.getMeasuredWidth();
        final int h = child.getMeasuredHeight();
        final int childTop = flowDown ? y : y - h;

        if (needToMeasure) {
            final int childRight = childrenLeft + w;
            final int childBottom = childTop + h;
            child.layout(childrenLeft, childTop, childRight, childBottom);
        }

```

第一次布局时，四个参数的值分别如下： **childrenLeft：**为ListView的PaddingLeft **childTop：**最初是为ListView的PaddingTop，之后为之前的一个View的底部的位置加上分隔线的高度。 **childRight：**为ListView的PaddingLeft加上View的宽度 **childBottom：**为ListView的PaddingTop加上View的高度

# 第二次测量

第二次测量与第一次测量最大的不同就是第二次测量的时候，ListView已经拥有了子View，在各逻辑的判断上回有所不同。 第二次测量与第一次其实查找的步骤差不多，首先我们还是要进入关键函数layoutChild（）查看，由于之前已经分析过滤过，所以此处就 贴上关键代码：

```
if (childCount == 0) {
                    if (!mStackFromBottom) {
                        final int position = lookForSelectablePosition(0, true);
                        setSelectedPositionInt(position);
                        sel = fillFromTop(childrenTop);
                    } else {
                        final int position = lookForSelectablePosition(mItemCount - 1, false);
                        setSelectedPositionInt(position);
                        sel = fillUp(mItemCount - 1, childrenBottom);
                    }
                } else {
                    if (mSelectedPosition &gt;= 0 &amp;&amp; mSelectedPosition &lt; mItemCount) {
                        sel = fillSpecific(mSelectedPosition,
                                oldSel == null ? childrenTop : oldSel.getTop());
                    } else if (mFirstPosition &lt; mItemCount) {
                        sel = fillSpecific(mFirstPosition,
                                oldFirst == null ? childrenTop : oldFirst.getTop());
                    } else {
                        sel = fillSpecific(0, childrenTop);
                    }
                }

```

之前我们分析这边的时候是默认子View的个数为0，所以进入的前面的语句，但是第二次测量的时候子View已经放入ListView中，因此childCount 不再等于0，于是我们会进入后面的语句。 我们还是默认是从上至下的默认排序，于是我们会进入fillSpecific()这个函数：

```
/**
     * Put a specific item at a specific location on the screen and then build
     * up and down from there.
     *
     * @param position The reference view to use as the starting point
     * @param top Pixel offset from the top of this view to the top of the
     *        reference view.
     *
     * @return The selected view, or null if the selected view is outside the
     *         visible area.
     */
    private View fillSpecific(int position, int top) {
        boolean tempIsSelected = position == mSelectedPosition;
        View temp = makeAndAddView(position, top, true, mListPadding.left, tempIsSelected);
        // Possibly changed again in fillUp if we add rows above this one.
        mFirstPosition = position;

        View above;
        View below;

        final int dividerHeight = mDividerHeight;
        if (!mStackFromBottom) {
            above = fillUp(position - 1, temp.getTop() - dividerHeight);
            // This will correct for the top of the first view not touching the top of the list
            adjustViewsUpOrDown();
            below = fillDown(position + 1, temp.getBottom() + dividerHeight);
            int childCount = getChildCount();
            if (childCount &gt; 0) {
                correctTooHigh(childCount);
            }
        } else {
            below = fillDown(position + 1, temp.getBottom() + dividerHeight);
            // This will correct for the bottom of the last view not touching the bottom of the list
            adjustViewsUpOrDown();
            above = fillUp(position - 1, temp.getTop() - dividerHeight);
            int childCount = getChildCount();
            if (childCount &gt; 0) {
                 correctTooLow(childCount);
            }
        }

        if (tempIsSelected) {
            return temp;
        } else if (above != null) {
            return above;
        } else {
            return below;
        }
    }

```

通过注释和逻辑我们可以知道，fillSpecific（）这个函数与fillDown（）和fillUp（）这两个函数其实差不多。只是fillSpecific（）会先加载指定的View，然后再往上往下加载其他的View。 接着我们就查看一下传过来的参数：

```
sel = fillSpecific(mSelectedPosition,oldSel == null ? childrenTop : oldSel.getTop());

```

```
fillSpecific(int position, int top)

```

当我们第一次加载的时候，由于我们还没有选择任何一项，所以mSelectedPosition的值为0。同样的我们也没有滑动过ListView，因此oldSel=null。于是我们可得，position=0，top=0。 而我们在fillSpecific()函数中进行的操作如下：

```
            below = fillDown(position + 1, temp.getBottom() + dividerHeight);
            // This will correct for the bottom of the last view not touching the bottom of the list
            adjustViewsUpOrDown();
            above = fillUp(position - 1, temp.getTop() - dividerHeight);
            int childCount = getChildCount();
            if (childCount &gt; 0) {
                 correctTooLow(childCount);

```

于是我们可以发现，其实我们还只是执行了一遍fillDown（）函数而已，只是第一次测量是通过fillFromTop()这个函数进入的，而第二次是通过fillSpecific()这个函数进入的。 之后的逻辑就是与第一次测量类似了。

# 总结（两次测量比较）

## 第一次测量

第一次测量的时候ListView中没有子View。 查找到layout关键代码步骤如下： **onLayout**(AbsListView类) **layoutChildren**(ListView类) **fillFromTop**(ListView类) **fillDown**(ListView类) **makeAndAddView**(ListView类) **setupChild**(ListView类)

## 第二次测量

第二次测量的时候ListView中已经拥有了子View。 查找到layout关键代码步骤如下： **onLayout**(AbsListView类) **layoutChildren**(ListView类) **fillSpecific**(ListView类) **fillDown**(ListView类) **makeAndAddView**(ListView类) **setupChild**(ListView类)

# 拓展

当然我相信有的读者的思维已经难以维持在这一点点的对ListView的理解了，本篇文章中只是阐述了ListView的两次测量的问题，而还有残留了较多的其他问题，比如：
1. ListView内部是如何进行子View的复用的呢？（使用RecycleBin）1. 既然ListView两次测量最终都会进入setupChild（）这个函数，那么这两次测量到底有什么不同呢？（第一次将数据放入RecycleBin，第二次直接从里面获取）
由于笔者阐述能力有限，这些问题就留给有兴趣的读者自己去进行源码的探索吧。

路漫漫其修远兮，吾将上下而求索。