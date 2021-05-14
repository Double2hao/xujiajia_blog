#Android Fragment事件透传
# 概述

近期使用Fragment，需要将事件透传到下层（可能是Activity，也可能是其他Fragment），于是作此文记录之。

原理其实很简单，就是让Fragment布局中ViewGroup都会将触摸事件传递到下层。主要是通过重写dispatchTouchEvent方法来实现。

需要注意的是，如果Fragment布局中嵌套较多，那么每一层都需要重写dispatchTouchEvent这个方法。

# demo

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040159770.png" alt="在这里插入图片描述">

# 代码

### MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

    private TestFragment fragment;
    private Button btn;

    public static int count = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {<!-- -->
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //设置fragment
        fragment = new TestFragment();
        FragmentManager fragmentManager = getSupportFragmentManager();
        FragmentTransaction ft = fragmentManager.beginTransaction();
        ft.add(android.R.id.content, fragment, "TestFragment");
        ft.commit();

        //设置button
        btn = findViewById(R.id.btn_main);
        btn.setOnClickListener(new View.OnClickListener() {<!-- -->
            @Override
            public void onClick(View v) {<!-- -->
                count ++;
                fragment.setTextTwo("MainActivity.btn点击次数："+count);
            }
        });
    }

}

```

### activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"&gt;

    &lt;Button
        android:id="@+id/btn_main"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="add count" /&gt;

&lt;/LinearLayout&gt;

```

### TestFragment.java

```
public class TestFragment extends Fragment {<!-- -->

    //ui
    private LinearLayout linearLayout;
    private TextView mTextViewOne;
    private TextView mTextViewTwo;

    //data
    private int count = 0;

    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {<!-- -->
        //将事件传递给Activity中的btn
        linearLayout = new LinearLayout(getContext()){<!-- -->
            @Override
            public boolean dispatchTouchEvent(MotionEvent ev) {<!-- -->
                super.dispatchTouchEvent(ev);
                return false;
            }
        };
        linearLayout.setOrientation(LinearLayout.VERTICAL);
        linearLayout.setBackgroundColor(getResources().getColor(R.color.mBlack));

        mTextViewOne = new TextView(getContext());
        mTextViewOne.setTextSize(50);
        mTextViewOne.setTextColor(Color.WHITE);
        linearLayout.addView(mTextViewOne);

        mTextViewTwo = new TextView(getContext());
        mTextViewTwo.setTextSize(50);
        mTextViewTwo.setTextColor(Color.WHITE);
        linearLayout.addView(mTextViewTwo);


        mTextViewTwo.setText("MainActivity.btn点击次数："+count);
        mTextViewOne.setText("fragment点击次数："+count);
        linearLayout.setOnClickListener(new View.OnClickListener() {<!-- -->
            @Override
            public void onClick(View v) {<!-- -->
                count++;
                mTextViewOne.setText("fragment点击次数："+count);
            }
        });

        return linearLayout;
    }

    public void setTextTwo(String text) {<!-- -->
        if (mTextViewTwo != null) {<!-- -->
            mTextViewTwo.setText(text);
        }
    }

}

```