#Android单元测试-对View的测试
# 前提概要

前两篇文章分别介绍了单元测试的作用和关于Activity的单元测试，有兴趣或者刚入门的读者可以阅读以下前两篇文章：    

这篇文章对应Activity中View的测试做一个专门的拓展。

# 主要三个组件（类）

### ViewMatchers

这个类中有对View的各种属性的判断方法，比如可以判断id，判断是否获取焦点，text内容为什么之类。  主要有两种作用，一是可以通过它获取到界面中的一个特定的View，二是可以通过它对获取到的View进行属性的断言，判断View的属性是否正确。

### ViewActions

这个对象能让View进行各种操作，比如点击操作，长按操作等。  主要作用是查看View在进行某种操作后是否执行了正确的逻辑。

### ViewAssertions

这个对象主要是和ViewMatchers 配合使用，用来判断获取到的View的属性是否正确。比如text是否与预期的相等，当下是否是可见的。

# 基本使用示例

<img src="https://img-blog.csdn.net/20170816165504974?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述" title="">  如图主要有一个TextView和一个ListView，测试代码如下：

```
public class MainActivity extends AppCompatActivity {<!-- -->

    private TextView tvMain;
    private ListView lvMain;
    private final String[] listString= {
        "11111","22222","33333","44444","55555"
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        tvMain=(TextView)findViewById(R.id.tv_main);
        lvMain=(ListView)findViewById(R.id.lv_main);

        tvMain.setText("mainTextView");
        lvMain.setAdapter(new ArrayAdapter&lt;&gt;(this,R.layout.item_text_view,listString));
        lvMain.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView&lt;?&gt; adapterView, View view, int i, long l) {
                Toast.makeText(MainActivity.this, ((TextView)view).getText(), Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

```
public class viewTest {<!-- -->
    @Rule
    public ActivityTestRule&lt;MainActivity&gt; mActivityRule =
            new ActivityTestRule&lt;&gt;(MainActivity.class);

    @Test
    public void testTextView(){
       onView(withId(R.id.tv_main))
               .check(matches(withText("mainTextView")));
    }

    @Test
    public void testListView(){
        onView(allOf(withId(R.id.tv_test),withText("22222")))
                .perform(click());

    }
}
```

TextView主要是查找到这个TextView的id，然后对比一下它的Text是否是“mainTextView”。如果不是的话Test会直接fail。

ListView的测试中如果只通过id查找显然是不合适的，因为listView中的每一个item都是相同的布局，所以可以通过allOf（）这个方法进行多个条件的查找，我们查找到这个item的id下的text为“22222”的View，然后再执行点击操作。

# 功能拓展

三个组件使用非常简单，但是功能也不仅如此，每个类的可使用性都很强，如下罗列各个方法的Reference的链接供读者参考。

### ViewMatchers



### ViewActions



### ViewAssertions

