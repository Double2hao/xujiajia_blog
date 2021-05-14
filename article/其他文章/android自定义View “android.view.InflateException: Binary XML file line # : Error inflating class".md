#android自定义View “android.view.InflateException: Binary XML file line # : Error inflating class"
<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209909808320.png " alt="这里写图片描述">

# 异常：

>  
 Caused by: android.view.InflateException: Binary XML file line #82: Error inflating class com.example.new_smartoffice.view.ThreeButtonItem 


笔者异常如上，具体意思大概就是说XML布局的82行有错，错误就出在笔者的自定义布局上面。但是笔者检查了代码好几遍，发现没有什么特殊的逻辑错误，于是纠结了好久，最后百度得发现是**构造函数**出了问题。

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209909810931.png " alt="这里写图片描述">

如上，一个ViewGroup的构造函数有四种，经笔者实践，在笔者例子中只有使用**第二个构造函数**才能够正常的运行。（笔者是重写的LinearLayout）

笔者例子中**正确的构造函数**应该如下：

```
public ThreeButtonItem(Context context, AttributeSet attrs) {
        super(context, attrs);

    }

```