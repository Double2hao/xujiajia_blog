#使用RecyclerView解决滑动冲突（RecyclerView添加header）
# 概述

在android平时开发中，经常会碰到同一个页面中有多个list的问题，或者需要再一个list中存在静态布局需要与list一起滚动的的需求。 针对这些情况，笔者尝试过多种方法，如：scrollView嵌套recyclerView，recyclerView嵌套recyclerView等。（主要工作量是在对分发事件的控制）

最终还是认为使用单独的一个recyclerView最为实用，主要优势如下：
1. 减少嵌套层级1. 减少对各种分发事件的控制
此文用简单demo记之。

>  
 笔者认为单独使用一个recyclerView最为实用，并不是说任何场景都要用这个。如下两个场景就不推荐使用： 
 - 很多时候，一个页面中的内容由多个模块共同拼凑而成，无法只由一个RecyclerView来写。这种场景还是需要活学活用，使用最方便、风险最小的方式实现。- 需要给其中一整块内容添加动画。这种情况最方便的解决方式还是通过增加层级实现，如果给每个Item都设置一下，反而显得得不偿失。 


# Demo

此demo主要以“RecyclerView添加header”为例来表示如何在recyclerView中实现复杂布局，要点如下：
1. 此demo是gridLayout，因此需要自定义GridLayoutManager，在第一行的时候只展示一列。1. Adapter提前在hedaer的位置放一个FrameLayout，这样设置header的时候只需要addView就可以。（通过这种方式可以让adapter更加灵活，header的创建与adapter代码也可以解耦）1. adapter中onCreateViewHolder和onBindViewHolder中要注意根据itemViewType来区分view的类型，根据不同的view设置。（此demo中，只需判断是header还是正常item）
# 代码

## MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

  //data
  private MainRecyclerViewAdapter adapterMain;
  //ui
  private RecyclerView rvMain;
  private View llHeader;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initViews();
    initData();
  }

  private void initViews() {<!-- -->
    rvMain = findViewById(R.id.rv_main);
    rvMain.setLayoutManager(new MainRecyclerViewAdapter.GridLayoutManager(this));

    llHeader = LayoutInflater.from(this).inflate(R.layout.view_recycler_header_main, null);
  }

  private void initData() {<!-- -->
    //设置测试数据
    List&lt;String&gt; listData = new ArrayList&lt;&gt;(300);
    for (int i = 0; i &lt; 300; i++) {<!-- -->
      listData.add(String.valueOf(i));
    }
    adapterMain = new MainRecyclerViewAdapter(this, rvMain);
    rvMain.setAdapter(adapterMain);
    adapterMain.setDataList(listData);
    adapterMain.setHeader(llHeader);
  }
}

```

## MainRecyclerViewAdapter.java

```
public class MainRecyclerViewAdapter
    extends RecyclerView.Adapter&lt;MainRecyclerViewAdapter.ViewHolder&gt; {<!-- -->
  //viewType
  private final static int VIEW_TYPE_HEADER = 0;
  private final static int VIEW_TYPE_DATA = 1;

  //data
  private Activity mActivity;
  private RecyclerView mRecyclerView;
  private List&lt;String&gt; dataList;
  //ui
  private FrameLayout headerContainer;

  public MainRecyclerViewAdapter(Activity activity, RecyclerView recyclerView) {<!-- -->
    mActivity = activity;
    mRecyclerView = recyclerView;

    createHeaderContainer();
  }

  private void createHeaderContainer() {<!-- -->
    headerContainer = new FrameLayout(mActivity);
    ViewGroup.LayoutParams lp = new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
        ViewGroup.LayoutParams.WRAP_CONTENT);
    headerContainer.setLayoutParams(lp);
  }

  public void setHeader(View header) {<!-- -->
    if (headerContainer == null) {<!-- -->
      return;
    }
    headerContainer.removeAllViews();
    headerContainer.addView(header);
  }

  public void setDataList(List&lt;String&gt; dataList) {<!-- -->
    this.dataList = dataList;
    notifyDataSetChanged();
  }

  @NonNull
  @Override
  public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {<!-- -->
    View itemView;
    switch (viewType) {<!-- -->
      case VIEW_TYPE_HEADER:
        itemView = headerContainer;
        break;
      default:
        itemView = new TextView(mActivity);
        itemView.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT,
            ViewGroup.LayoutParams.WRAP_CONTENT));
        break;
    }
    return new ViewHolder(itemView);
  }

  @Override public int getItemViewType(int position) {<!-- -->
    if (position == 0) {<!-- -->
      return VIEW_TYPE_HEADER;
    }
    return VIEW_TYPE_DATA;
  }

  @Override
  public void onBindViewHolder(@NonNull ViewHolder holder, int position) {<!-- -->
    if (position == 0) {<!-- -->
      return;
    }

    String nowData = dataList.get(position - 1);
    ((TextView) holder.itemView).setText(nowData);
  }

  @Override
  public int getItemCount() {<!-- -->
    if (dataList == null) {<!-- -->
      return 1;
    }
    return dataList.size() + 1;
  }

  //span count
  private final static int HEADER_SPAN_COUNT = 1;
  private final static int DATA_SPAN_COUNT = 3;

  public static class ViewHolder extends RecyclerView.ViewHolder {<!-- -->

    public ViewHolder(@NonNull View itemView) {<!-- -->
      super(itemView);
    }
  }

  public static class GridLayoutManager extends androidx.recyclerview.widget.GridLayoutManager {<!-- -->

    public GridLayoutManager(Context context) {<!-- -->
      super(context, DATA_SPAN_COUNT);

      this.setSpanSizeLookup(new SpanSizeLookup() {<!-- -->
        @Override public int getSpanSize(int position) {<!-- -->
          return position == 0 ? DATA_SPAN_COUNT : HEADER_SPAN_COUNT;
        }
      });
    }
  }
}


```

## activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    &gt;

  &lt;TextView
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:gravity="center"
      android:text="RecyclerView测试"
      android:textSize="30sp"
      /&gt;

  &lt;androidx.recyclerview.widget.RecyclerView
      android:id="@+id/rv_main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      /&gt;

&lt;/LinearLayout&gt;

```

## view_recycler_header_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    &gt;
  &lt;Button
      android:layout_width="match_parent"
      android:layout_height="200dp"
      android:text="这是内容1"
      /&gt;
&lt;/LinearLayout&gt;

```