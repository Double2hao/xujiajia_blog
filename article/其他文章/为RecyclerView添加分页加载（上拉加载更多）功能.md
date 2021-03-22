#为RecyclerView添加分页加载（上拉加载更多）功能
上一篇文章已经介绍了如何为RecyclerView添加FootView，在此基础上，要添加分页加载的功能其实已经很简单了。 上一篇文章地址：

# 效果：（源码在文章结尾）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2400.png" alt="这里写图片描述">

# 实现关键

在上一篇代码的基础上，只需要在onBindViewHolder(ViewHolder holder, int position)函数中添加一定修改就可以了，如下：

```
@Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        //建立起ViewHolder中试图与数据的关联
        Log.d("xjj", getItemViewType(position) + "");
        //如果footview存在，并且当前位置ViewType是FOOT_TYPE
        if (isFootView &amp;&amp; (getItemViewType(position) == FOOT_TYPE)) {
            holder.tvFootView.setText(footViewText);
            // 刷新太快 所以使用Hanlder延迟两秒
            Handler handler = new Handler();
            handler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    max_count += 5;
                    notifyDataSetChanged();
                }
            }, 2000);

        } else {
            holder.tvViewHolder.setText(mData.get(position) + position);
        }
    }

```

在函数中，首先让该item显示“加载中。。。”，然后使用Handler，延迟两秒刷新，逻辑内容主要有两个，一个是显示的最大容量增加5，二是刷新Adapter的内容。

# 拓展延伸

笔者此处为了让读者容易理解，很多地方的使用比较粗糙，读者真正使用的时候定然不会如此简单,在此列出几点，以供读者自己学习： 1、FootView中一般不会仅仅是一个TextView，对UI有一定追求的读者至少需要添加一个ProgressBar。 2、Adapter的内容一般不会直接用List传入，需要从网络获取之类，因此刷新Adapter的内容一般就需要用到线程，而不是此处简单的一个Handler就可以了。

源码地址：