#getView()不复用convertView，ListView即毫无复用！（ListView回收机制）
#前提概要

>  
 getView()不复用convertView，ListView即毫无复用！ 


标题的话不知道有没有震惊到大家，反正是与笔者一直以来的想法相悖的。 之前笔者一直认为，使用BaseAdapter如果不复用convertView，那么可能会一定程度上降低ListView的性能，但是ListView本身一定还有其优化性能的方式，所以写demo时也经常偷一下懒，就不写convertView的复用了。 但是却绝对想不到，如果getView中不复用convertView，ListView本身的复用机制就几乎毫无作用了。 #ObtainView（） ListView内部对于View的获取的优化其实主要还是通过ObtainView（），这个方法在ListView中是查看不到的，这是它的父类AbsListView的方法。

```
    View obtainView(int position, boolean[] outMetadata) {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "obtainView");

        outMetadata[0] = false;

        // Check whether we have a transient state view. Attempt to re-bind the
        // data and discard the view if we fail.
        final View transientView = mRecycler.getTransientStateView(position);
        if (transientView != null) {
            final LayoutParams params = (LayoutParams) transientView.getLayoutParams();

            // If the view type hasn't changed, attempt to re-bind the data.
            if (params.viewType == mAdapter.getItemViewType(position)) {
                final View updatedView = mAdapter.getView(position, transientView, this);

                // If we failed to re-bind the data, scrap the obtained view.
                if (updatedView != transientView) {
                    setItemViewLayoutParams(updatedView, position);
                    mRecycler.addScrapView(updatedView, position);
                }
            }

            outMetadata[0] = true;

            // Finish the temporary detach started in addScrapView().
            transientView.dispatchFinishTemporaryDetach();
            return transientView;
        }

        final View scrapView = mRecycler.getScrapView(position);
        final View child = mAdapter.getView(position, scrapView, this);
        if (scrapView != null) {
            if (child != scrapView) {
                // Failed to re-bind the data, return scrap to the heap.
                mRecycler.addScrapView(scrapView, position);
            } else if (child.isTemporarilyDetached()) {
                outMetadata[0] = true;

                // Finish the temporary detach started in addScrapView().
                child.dispatchFinishTemporaryDetach();
            }
        }

        if (mCacheColorHint != 0) {
            child.setDrawingCacheBackgroundColor(mCacheColorHint);
        }

        if (child.getImportantForAccessibility() == IMPORTANT_FOR_ACCESSIBILITY_AUTO) {
            child.setImportantForAccessibility(IMPORTANT_FOR_ACCESSIBILITY_YES);
        }

        setItemViewLayoutParams(child, position);

        if (AccessibilityManager.getInstance(mContext).isEnabled()) {
            if (mAccessibilityDelegate == null) {
                mAccessibilityDelegate = new ListItemAccessibilityDelegate();
            }
            if (child.getAccessibilityDelegate() == null) {
                child.setAccessibilityDelegate(mAccessibilityDelegate);
            }
        }

        Trace.traceEnd(Trace.TRACE_TAG_VIEW);

        return child;
    }

```

代码比较长，但是并不是所有的都要看，transientView是Listview内部对带动画的View的一些复用处理，我们可以暂时避开它不看，主要的逻辑其实是scrapView那一块。

```
        final View scrapView = mRecycler.getScrapView(position);
        final View child = mAdapter.getView(position, scrapView, this);
        if (scrapView != null) {
            if (child != scrapView) {
                // Failed to re-bind the data, return scrap to the heap.
                mRecycler.addScrapView(scrapView, position);
            } else if (child.isTemporarilyDetached()) {
                outMetadata[0] = true;

                // Finish the temporary detach started in addScrapView().
                child.dispatchFinishTemporaryDetach();
            }
        }
        .........................................（此处代码省略）
        return child;

```

此处逻辑主要如下： 1、从回收站中获取到要复用的View，即scrapView 。 2、执行adapter的getView操作，并且获取到View，即child。 3、如果存在复用的View，那么就查看一下getView()，返回的结果与它是不是同一个对象，如果是同一个对象，那么代表复用成功（getView中仅仅修改了数据）。如果不是同一个对象，代表没有复用，把这个复用的View重新放回回收站中。

此处的第二个参数scrapView 其实就是我们getView方法中的参数convertView。 我们可以看到，ListView的复用方式主要是通过判断convertView与getView的返回对象是否是同一个对象，如果是就复用，如果不是就不复用。 倘若我们的getView()方法仅仅为了方便每次创建新的View，而没有复用convertView，那么必然ListView就不会复用了。 #SimpleAdapter的getView（） 当然一般我们会重写getView只是在BaseAdapter中，那么其他的官方所写的ListView使用的Adatper是不是就复用了呢？ 我们可以看一下SimpleAdapter的getView（）方法：

```
    public View getView(int position, View convertView, ViewGroup parent) {
        return createViewFromResource(mInflater, position, convertView, parent, mResource);
    }

```

```
 private View createViewFromResource(LayoutInflater inflater, int position, View convertView,
            ViewGroup parent, int resource) {
        View v;
        if (convertView == null) {
            v = inflater.inflate(resource, parent, false);
        } else {
            v = convertView;
        }

        bindView(position, v);

        return v;
    }

```

如上代码所示，我们可以看到，在需求是“每个Item布局相同”的情况下，的确是通过返回convertView来保证ListView的复用。

#ViewHolder 到此处，好奇的读者可能又会想到“那么ViewHolder在ListView的复用中又是担任什么样的一个角色呢？”。 我们且看我们复用时getView()的demo代码：

```
    static class ViewHolder {
        public ImageView img;
        public TextView title;
        public TextView info;
    }
    
  @Override
    public View getView(int position, View convertView, ViewGroup parent) {
// TODO Auto-generated method stub
        ViewHolder holder = null;
        if (convertView == null) {
            holder = new ViewHolder();
            convertView = mInflater.inflate(R.layout.list_item, null); //这里确定你listview里面每一个item的layout
            holder.img = (ImageView) convertView.findViewById(R.id.img); //此处是将内容与控件绑定。

            holder.title = (TextView) convertView.findViewById(R.id.tv);//注意：此处的findVIewById前要加convertView.
            holder.info = (TextView) convertView.findViewById(R.id.info);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag(); //这里是为了提高listview的运行效率
        }
        holder.img.setBackgroundResource((Integer) data.get(position).get("img"));
        holder.title.setText((String) data.get(position).get("title"));
        holder.info.setText((String) data.get(position).get("info"));

        return convertView;
    }

```

这个示例中，我们可以看到，最终返回的是convertView，所以ListView的复用机制是成立的。所以这就避免了View的多次创建，即不会每次都执行如下代码创建View：

```
convertView = mInflater.inflate(R.layout.list_item, null);

```

而同时也使用setTag（）的方法，将自定义的ViewHolder放入了，这不是针对“创建View”的优化，而是针对“获取View”的优化，使用了ViewHolder之后，就不需要每次都去执行从convertView中findViewById的操作了。

当然这一切的前提是Listview即将创建的布局与复用的布局相同的情况，或者说仅仅只有数据不同的情况。

##文章如有谬误，欢迎评论指正。