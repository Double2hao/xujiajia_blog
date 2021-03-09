#为RecyclerView添加FootView和HeadView
# 前提概要：

上一篇文章已经介绍过了RecyclerView的基本使用方法，原文如下：此篇文章算是对RecyclerView更深使用的介绍。

FootView和HeadView在ListView中的本身就有相对应的函数，但是在新潮的RecyclerView中却没有了，FootView在分页加载（上拉加载更多）中起着很重要的作用，因此也必须要学习一下了。（HeadView的添加与FootView的添加相比大致一样，在此就只讲FootView的添加了）

# 效果：（源码在文章结尾）

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDExMTQxNTIwNjE0" alt="这里写图片描述">

# 实现关键

**int getItemViewType(int position)**：此函数是RecyclerView中自带的函数，参数为每个item的position，返回一个int类型表示类型。

此函数的作用是**区分普通的item与FootView的Item**，让FootView这个Item能一直处在adapter中的最下端。

在例子中定义了两种类型如下：

```
 //两个final int类型表示ViewType的两种类型
    private final int NORMAL_TYPE = 0;
    private final int FOOT_TYPE = 1111;

```

该函数如下：

```
    @Override
    public int getItemViewType(int position) {
        if (position == max_count - 1) {
            return FOOT_TYPE;
        }
        return NORMAL_TYPE;
    }

```

# 实现步骤

1、定义**getItemViewType(int position)<strong>并且定义自己所需要的**ViewType</strong>的类型。 2、在定义**ViewHolder()**，**onCreateViewHolder()**和**onBindViewHolder()**中都考虑两种情况，一种是普通的item，另一种是FootView。 另外默认的ViewHolder()函数中只会有View itemView一个参数，此处因为需要，所以要添加int viewType的参数，代码中如下：

```
//初始化viewHolder，此处绑定后在onBindViewHolder中可以直接使用
        public ViewHolder(View itemView, int viewType) {
            super(itemView);
            if (viewType == NORMAL_TYPE) {
                tvViewHolder = (TextView) itemView.findViewById(R.id.tv_view_holder);
                llViewHolder = (LinearLayout) itemView;
            } else if (viewType == FOOT_TYPE) {
                tvFootView = (TextView) itemView;
            }
        }

```

此处再说一下**三个函数的大致作用**：

## ViewHolder(View itemView, int viewType):

将item布局中的控件与ViewHolder中所定义的属性绑定，更便于在**onBindViewHolder()**中使用。

## onCreateViewHolder(ViewGroup parent, int viewType)：

此函数用来创建每一个item，最后返回的不是view，而是返回的一个ViewHolder。

## onBindViewHolder(ViewHolder holder, int position)：

此函数中一般用来将数据绑定到item中的控件中。

# 代码：

结合以上分析看代码，读者应该比较容易理解了，下面附上关键adapter代码和源码地址：

```
public class RecyclerAdapter extends RecyclerView.Adapter&lt;RecyclerAdapter.ViewHolder&gt; {
    private List&lt;String&gt; mData;//数据
    private int max_count = 10;//最大显示数
    private Boolean isFootView = false;//是否添加了FootView
    private String footViewText = "";//FootView的内容

    //两个final int类型表示ViewType的两种类型
    private final int NORMAL_TYPE = 0;
    private final int FOOT_TYPE = 1111;


    public RecyclerAdapter(List&lt;String&gt; data) {
        mData = data;
    }

    public class ViewHolder extends RecyclerView.ViewHolder {
        public TextView tvViewHolder;
        public LinearLayout llViewHolder;

        public TextView tvFootView;//footView的TextView属于独自的一个layout

        //初始化viewHolder，此处绑定后在onBindViewHolder中可以直接使用
        public ViewHolder(View itemView, int viewType) {
            super(itemView);
            if (viewType == NORMAL_TYPE) {
                tvViewHolder = (TextView) itemView.findViewById(R.id.tv_view_holder);
                llViewHolder = (LinearLayout) itemView;
            } else if (viewType == FOOT_TYPE) {
                tvFootView = (TextView) itemView;
            }
        }
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View normal_views = LayoutInflater.from(parent.getContext()).inflate(
                R.layout.rc_item, parent, false);
        View foot_view = LayoutInflater.from(parent.getContext()).inflate(
                R.layout.foot_view, parent, false);

        if (viewType == FOOT_TYPE)
            return new ViewHolder(foot_view, FOOT_TYPE);
        return new ViewHolder(normal_views, NORMAL_TYPE);
    }

    @Override
    public int getItemViewType(int position) {
        if (position == max_count - 1) {
            return FOOT_TYPE;
        }
        return NORMAL_TYPE;
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        //建立起ViewHolder中试图与数据的关联
        Log.d("xjj", getItemViewType(position) + "");
        //如果footview存在，并且当前位置ViewType是FOOT_TYPE
        if (isFootView &amp;&amp; (getItemViewType(position) == FOOT_TYPE)) {
            holder.tvFootView.setText(footViewText);
        } else {
            holder.tvViewHolder.setText(mData.get(position) + position);
        }
    }

    @Override
    public int getItemCount() {
        if (mData.size() &lt; max_count) {
            return mData.size();
        }
        return max_count;
    }

    //创建一个方法来设置footView中的文字
    public void setFootViewText(String footViewText) {
        isFootView = true;
        this.footViewText = footViewText;
    }
}

```

# 拓展延伸

下一篇文章中，笔者在此基础上做了一下一点改动，很容易就实现了分页加载的功能，文章地址如下： 

源码地址：
