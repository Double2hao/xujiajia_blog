#android SharedPreference的简单使用（登陆界面记住密码）
SharedPreference方面的内容还算是比较简单易懂的，在此还是主要贴上效果与代码，最后也是附上源码。

 

首先是输入账号admin，密码123，选择记住密码登陆。

登陆后就直接跳转页面。

 

<img alt="" class="has" height="500" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910559783.png " width="300">         <img alt="" class="has" height="500" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910554961.png " width="300">

 

 

 

随后再次打开app可以发现已经记住了密码。

 

如果输错了密码，或者在登陆的时候没有选择记住密码，那么会将SharedPreference的文件清空，再次登陆后又会是空的EditText。

 

<img alt="" class="has" height="500" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910557222.png " width="300">       <img alt="" class="has" height="500" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910559783.png " width="300">

 

 

 

MainActivity:

 

```
import android.app.Activity;
import android.content.Intent;
import android.content.SharedPreferences;
import android.preference.PreferenceManager;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.Window;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.Toast;

public class MainActivity extends Activity {

    //使用SharedPreferences进行读取
    private SharedPreferences pref;
    //使用SharedPreferences.Editor进行存储
    private SharedPreferences.Editor editor;

    private Button button;
    private CheckBox checkbox;
    private EditText accountEdit;
    private EditText passwordEdit;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //取消标题
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_main);

        //制定SharedPreference的文件名为data
        pref= getSharedPreferences("data",MODE_PRIVATE);
        editor=pref.edit();
        accountEdit=(EditText)findViewById(R.id.account);
        passwordEdit=(EditText)findViewById(R.id.password);
        checkbox=(CheckBox)findViewById(R.id.remember_password);
        button=(Button)findViewById(R.id.button);

        //查看app中是否已经存储过账号密码，有的话就直接显示出来
        boolean isRemember=pref.getBoolean("remember_password",false);
        if(isRemember){
            String account=pref.getString("account","");
            String password=pref.getString("password","");
            accountEdit.setText(account);
            passwordEdit.setText(password);
            checkbox.setChecked(true);
        }


        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                String account=accountEdit.getText().toString();
                String password=passwordEdit.getText().toString();
                //此处设置的账号为“admin”，密码为“123”
                if(account.equals("admin")&amp;&amp;password.equals("123")){
                    //如果记住密码被选中，那么就将账号密码存储起来
                    //如果没有被选中，那么将存储的账号密码清空
                    if(checkbox.isChecked()){
                        editor.putBoolean("remember_password",true);
                        editor.putString("account",account);
                        editor.putString("password",password);
                    }else{
                        editor.clear();
                    }
                    //最后千万不要忘记使用commit将添加的数据提交
                    editor.commit();

                    //简单的跳转界面
                    Intent intent=new Intent(MainActivity.this,NextActivity.class);
                    startActivity(intent);
                    finish();
                }else{
                    //如果账号密码错误，就跳出toast提示,并且清空存储的账号密码
                    Toast.makeText(MainActivity.this,"账号或密码错误！",Toast.LENGTH_LONG).show();

                    editor.clear();
                    editor.commit();
                }
            }
        });
    }


}
```

 

 

 

activity_main:

 

```
&lt;TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:stretchColumns="1"
    tools:context=".MainActivity"&gt;

    &lt;TableRow&gt;
        &lt;TextView
            android:textSize="20sp"
            android:text="账号:"
            android:textColor="#000"
            android:layout_height="wrap_content"/&gt;
        &lt;EditText
            android:id="@+id/account"
            android:hint="请输入你的账号"
            /&gt;
    &lt;/TableRow&gt;

    &lt;TableRow&gt;
        &lt;TextView
            android:textSize="20sp"
            android:text="密码:"
            android:textColor="#000"
            android:layout_height="wrap_content"/&gt;
        &lt;EditText
            android:id="@+id/password"
            android:hint="请输入你的密码"
            android:password="true"
            /&gt;
    &lt;/TableRow&gt;

    &lt;TableRow&gt;
        &lt;CheckBox
            android:id="@+id/remember_password"
            android:layout_height="wrap_content"
            android:layout_gravity="center"/&gt;
        &lt;TextView
            android:textSize="20sp"
            android:text="记住密码"
            android:layout_height="wrap_content"/&gt;

    &lt;/TableRow&gt;

    &lt;TableRow&gt;
        &lt;Button
            android:id="@+id/button"
            android:layout_span="2"
            android:text="确定"
            android:textSize="25sp"/&gt;
    &lt;/TableRow&gt;

&lt;/TableLayout&gt;
```

 

 NextActivity:

 

 

 

```
import android.app.Activity;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.Window;

public class NextActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_next);
    }


}
```

 

 

 

 

 

activity_next:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center_vertical"&gt;

    &lt;TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="登陆成功!"
        android:textSize="30sp"
        android:layout_gravity="center_horizontal"
        /&gt;
&lt;/LinearLayout&gt;
```

 

 

 

 

 

最后附上源码：