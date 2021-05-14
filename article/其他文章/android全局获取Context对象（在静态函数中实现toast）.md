#android全局获取Context对象（在静态函数中实现toast）
全局获取Context对象的意义：

当应用程序的架构逐渐复杂起来的时候，很多逻辑代码都将脱离Activity或者Service，但是如果此时你又恰恰需要使用Context，这个时候就需要用到全局获取Context了。

 

举个例子，如果此时需要实现一个从网络获取图片的类，我有多个Activity，并且每个Activity都需要从网络获取图片。在图片无法获得的时候，我需要用Toast来提示用户。那么看似一个简单的Toast此时却显得让人头疼了，在函数中如何能获取到Context对象了？换言之，我怎么知道我当前函数使用在哪个Activity中。

 

有两种方案可以实现：

1、在函数的构造函数中添加一个Context的参数，每次在使用函数的时候都同时导入当前使用的Activity。（但是此方案有点推卸责任的嫌疑，因为我们把获取Context的责任转移给了Toast的调用方，倘若调用方也是一个方法，并且没有Context的参数，那怎么办呢？）

**2、全局获取Context对象。此方法也正是本文所讲的方法。**

 

方法其实很简单，直接上图上代码：

<img alt="" class="has" height="700" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910453380.png " width="400">

 

MainActivity:

 

```
import android.app.Activity;
import android.content.DialogInterface;
import android.os.Bundle;
import android.support.design.widget.FloatingActionButton;
import android.support.design.widget.Snackbar;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.View;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends Activity implements View.OnClickListener {

    private Button button1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        button1=(Button)findViewById(R.id.toast1);
        button1.setOnClickListener(this);
    }

    @Override
    public void onClick(View view) {
        switch(view.getId()){
            case R.id.toast1:
                ToastShow1.ToastShow1();
                break;
        }
    }


    
}
```

 

 

ToastShow1:

 

```
import android.widget.Toast;

public class ToastShow1 {
    public static void ToastShow1() {
        Toast.makeText(MyApplication.getContext(),"Toast实现成功",Toast.LENGTH_LONG).show();
    }
}
```

 

 

MyApplication:

 

```
import android.app.Application;
import android.content.Context;

public class MyApplication extends Application {
    private  static Context context;

    @Override
    public void onCreate() {
        super.onCreate();
        context=getApplicationContext();
    }

    public static Context getContext() {
        return context;
    }
}
```

 

 

 

我们要告知系统，当程序启动的时候应该初始化MyApplication这个类，而不是系统默认的Application类。

这里在指定的MyApplication的时候一定要加上完整的包名，不然系统将无法找到这个类。

AndroidManifest:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.getcontexttest"&gt;

    &lt;application
        android:name="com.example.getcontexttest.MyApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"&gt;
        &lt;activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:theme="@style/AppTheme.NoActionBar"&gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
    &lt;/application&gt;

&lt;/manifest&gt;

```

 activity_main:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"&gt;

    &lt;Button
        android:id="@+id/toast1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Toast1" /&gt;
    
&lt;/LinearLayout&gt;

```

 

 

 

 

 

 

 

 

 

 

 

 

 

 