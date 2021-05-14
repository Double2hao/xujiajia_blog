#android一些常用基础UI控件（比较适合给新手参考）
近期又回到了基础，打算重新将郭霖前辈的《第一行代码》上的demo全部自己消化一遍，今天也是学完了UI一章，自己认真写了个demo，将一些常用的基础控件都写了进去，希望对部分新手能有帮助。

 

主要涉及到的内容有：

**TextView；**

**EditView包括其内容获取和默认状态字体设置，并涉及了Toast显示；**

**ProgreBar显示与不显示，进度条的设置；**

**AlertDialog的设置；**

 

还是先上图：

<img alt="" class="has" height="1000" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911192920.png " width="500">

 

 

 

MainActivity:

 

 

```
import android.app.Activity;
import android.app.AlertDialog;
import android.content.DialogInterface;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ProgressBar;
import android.widget.Toast;

public class MainActivity extends Activity implements View.OnClickListener {

    private EditText EditText;
    private Button EditText_button;

    private Button button1;
    private Button button2;
    private ProgressBar progressBar;

    private Button AlertDialog_button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        editTextInit();
        progressBarInit();
        AlertDialogInit();
    }

    private void editTextInit() {
        EditText=(EditText)findViewById(R.id.editText);
        EditText_button=(Button)findViewById(R.id.editText_button);
        EditText_button.setOnClickListener(this);
    }

    private void AlertDialogInit() {
        AlertDialog_button=(Button)findViewById(R.id.alertDialog_button);
        AlertDialog_button.setOnClickListener(this);
    }

    private void progressBarInit() {
        button1 = (Button) findViewById(R.id.progressBar_button1);
        button2 = (Button) findViewById(R.id.progressBar_button2);
        progressBar = (ProgressBar) findViewById(R.id.progress_bar);
        button1.setOnClickListener(this);
        button2.setOnClickListener(this);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.editText_button:
                //获取EditText中的内容
                String text=EditText.getText().toString();
                //用Toast来显示该内容
                Toast.makeText(MainActivity.this,text,Toast.LENGTH_LONG).show();
                break;

            //VISIBLE是可见的
            //INVISIBLE表示不可见，但是占据位置和大小
            //GONE表示不可见而且不占屏幕空间
            case R.id.progressBar_button1:
                if (progressBar.getVisibility() == View.GONE) {
                    progressBar.setVisibility(View.VISIBLE);
                } else {
                    progressBar.setVisibility(View.GONE);
                }
                break;


            case R.id.progressBar_button2:
                int progress = progressBar.getProgress();
                if (progress == 100)
                    progress = 0;
                else
                    progress = progress + 10;//每点击一次进度条增加10的进度
                progressBar.setProgress(progress);
                break;

            case R.id.alertDialog_button:
                AlertDialog.Builder dialog=new AlertDialog.Builder(MainActivity.this);
                dialog.setTitle("This is Dialog");
                dialog.setMessage("Something important.");
                dialog.setCancelable(false);
                dialog.setPositiveButton("Cancel", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialogInterface, int i) {
                            }
                        }
                );

                dialog.setNegativeButton("Ok", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialogInterface, int i) {
                            }
                        }
                );
                dialog.show();
                break;

        }
    }


}
```

 

 

 

 

activity_main:

 

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:orientation="vertical"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MainActivity"&gt;

    &lt;TextView
        android:textSize="20sp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/hello_world" /&gt;

    &lt;EditText
        android:id="@+id/editText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Type SomeThing here"
        android:maxLines="1"/&gt;

    &lt;Button
        android:id="@+id/editText_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="EditText"/&gt;

    &lt;Button
        android:id="@+id/progressBar_button1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ProgressBar1"/&gt;

    &lt;Button
        android:id="@+id/progressBar_button2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ProgressBar2"/&gt;
    &lt;Button
        android:id="@+id/alertDialog_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="AlertDialog"/&gt;

    &lt;ProgressBar
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/progress_bar"
        android:max="100"
         /&gt;




&lt;/LinearLayout&gt;
```

 

 

 

 

 

 

 