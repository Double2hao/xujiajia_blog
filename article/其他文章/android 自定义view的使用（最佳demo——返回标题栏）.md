#android 自定义view的使用（最佳demo——返回标题栏）
笔者近期在做项目时遇到了如下情况：有**超过10个**的子界面需要用到差不多的一个标题栏，如下图：

 

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1760.png" alt=""> 

 

倘若直接在每个布局文件中写一个这样的Linearlayout，一方面重复性的代码总是让人不太舒服，另一方面，我需要对同样的一个**返回键**在不同的布局中定义不同的id，然后还要在java文件中实现它的功能，实在是太让人不爽了。

 

于是，笔者就想到了**自定义布局**。

**自定义布局**的好处就是在布局定义好之后，直接可以在布局文件中使用，如下图： 

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1761.png" alt="">          <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1762.png" alt=""> 

 

不过由于每个布局的特殊性，所以还是需要一个方法——**设置textview的内容**，此处也是直接可以在代码中实现。

 

上一下项目文件：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1763.png" alt=""> 

 

MainActivity:



```
import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends Activity {

    private Button button;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        button=(Button)findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent=new Intent(MainActivity.this,TwoActivity.class);
                startActivity(intent);
                //此处千万外要注意，不要写finish()，在TwoActivity中按下返回键之后会摧毁TwoActivity的布局，
                //然后显示的就是MainActivity的布局，倘若把MainActivity的布局给finnish掉后，程序会直接退出
            }
        });
    }


}

```



 



```
import android.app.Activity;
import android.os.Bundle;

public class TwoActivity extends Activity{

    private TitleLayout title;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_two);
        //绑定TitleLayout
        //设置TitleLayout中text内容
        title=(TitleLayout)findViewById(R.id.title);
        title.setTitleText("Light_Control");
    }
}
```



 



```
import android.app.Activity;
import android.content.Context;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.TextView;

public class TitleLayout extends LinearLayout {

    private Button titleBack;
    private TextView titleText;

    public TitleLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        //加载布局文件，与setContentView()效果一样
        LayoutInflater.from(context).inflate(R.layout.my_view, this);
        titleBack = (Button) findViewById(R.id.title_back);
        titleText = (TextView) findViewById(R.id.title_text);

        //设置返回键的点击效果
        titleBack.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                ((Activity) getContext()).finish();
            }
        });

    }

    //创建一个方法来改变title中text的内容
    public void setTitleText(String text) {
        titleText.setText(text);
    }
}
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    &gt;

    &lt;Button
        android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="这是第一个Activity"
        android:textSize="30sp"
        android:gravity="center"/&gt;
&lt;/LinearLayout&gt;
```



 



```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;

    &lt;com.example.user_defined_view.TitleLayout
        android:id="@+id/title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"&gt;
    &lt;/com.example.user_defined_view.TitleLayout&gt;

    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="这是第二个Activity"
        android:textSize="30sp"/&gt;
&lt;/LinearLayout&gt;
```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="horizontal"
    android:layout_height="wrap_content"
    android:background="#272727"&gt;

    &lt;Button
        android:id="@+id/title_back"
        android:layout_width="45dp"
        android:layout_height="45dp"
        android:layout_gravity="center_vertical"
        android:background="@drawable/button_back"/&gt;
    &lt;TextView
        android:id="@+id/title_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="This is a Title"
        android:padding="12dp"
        android:layout_gravity="center_vertical"
        android:textSize="20sp"
        android:textStyle="bold"
        android:textColor="#fff"/&gt;
&lt;/LinearLayout&gt;
```



 

 