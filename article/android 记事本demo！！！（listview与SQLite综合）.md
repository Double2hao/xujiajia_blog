#android 记事本demo！！！（listview与SQLite综合）
android记事本的demo在网上一搜一大堆，**但是大神写的demo往往功能太多导致新手难以着手，很难啃得动；而一些新手写的demo又往往是东拼西凑，代码很多都是copy的别人的，直接放在项目里面用，也不知道代码有什么作用。往往代码特别丑，重复性的代码也比较多。**

笔者近期学到此处，自己理解之后也还是打算写个demo供新手学习一下。代码说不上优雅，但在笔者看来已经尽力去让人容易理解了。（源码在文章结尾）

 

为了便于新手学习，在此也是罗列一下涉及的知识点：

1、SQLite的基本使用，增删查改

2、listview，adapeter的基本使用

3、activity生命周期

4、intent、bundle传递参数

5、AlertDialog的基本使用

另外还有一些零碎知识点都可以百度到。

 

遇到的问题：

**SQlite有个问题，就是主键不能够自动排序。比如说主键id为1 2 3 4，共4条记录。现在删除2 3，还剩下1 4记录，当再次插入时，id会变成5，而不是2.假设在初始4条记录的基础上，把这4条记录全都删掉，再次插入时，得到的id是5.**

笔者在这点上也是花了比较久的时间，原本为了精简代码，想法是用listview中的arg2直接通过数据库记录的id进行操作，但是由于SQLite的这个问题，所以这种方法就有问题了。

最终，笔者采用的是内容搜索的方法，从listview的每个item中获取内容，然后到数据库中通过内容搜索该记录，最后对其进行操作。

 

 

效果：

<img alt="" class="has" height="530" src="https://img-blog.csdn.net/20151213192812968?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300"> <img alt="" class="has" height="530" src="https://img-blog.csdn.net/20151213192807910?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300"> <img alt="" class="has" height="530" src="https://img-blog.csdn.net/20151213192755976?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300">

 

 

**MainActivity：**

 

```
import android.app.Activity;
import android.app.AlertDialog.Builder;
import android.content.DialogInterface;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.AdapterView.OnItemLongClickListener;
import android.widget.Button;
import android.widget.ListView;
import android.widget.SimpleAdapter;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class MainActivity extends Activity implements
        OnItemClickListener, OnItemLongClickListener {

    private ListView listview;
    private SimpleAdapter simple_adapter;
    private List&lt;Map&lt;String, Object&gt;&gt; dataList;
    private Button addNote;
    private TextView tv_content;
    private NoteDateBaseHelper DbHelper;
    private SQLiteDatabase DB;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        InitView();
    }

    //在activity显示的时候更新listview
    @Override
    protected void onStart() {
        super.onStart();
        RefreshNotesList();
    }


    private void InitView() {
        tv_content = (TextView) findViewById(R.id.tv_content);
        listview = (ListView) findViewById(R.id.listview);
        dataList = new ArrayList&lt;Map&lt;String, Object&gt;&gt;();
        addNote = (Button) findViewById(R.id.btn_editnote);
        DbHelper = new NoteDateBaseHelper(this);
        DB = DbHelper.getReadableDatabase();

        listview.setOnItemClickListener(this);
        listview.setOnItemLongClickListener(this);
        addNote.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View arg0) {
                Intent intent = new Intent(MainActivity.this, noteEdit.class);
                Bundle bundle = new Bundle();
                bundle.putString("info", "");
                bundle.putInt("enter_state", 0);
                intent.putExtras(bundle);
                startActivity(intent);
            }
        });
    }


    //刷新listview
    public void RefreshNotesList() {
        //如果dataList已经有的内容，全部删掉
        //并且更新simp_adapter
        int size = dataList.size();
        if (size &gt; 0) {
            dataList.removeAll(dataList);
            simple_adapter.notifyDataSetChanged();
        }

        //从数据库读取信息
        Cursor cursor = DB.query("note", null, null, null, null, null, null);
        startManagingCursor(cursor);
        while (cursor.moveToNext()) {
            String name = cursor.getString(cursor.getColumnIndex("content"));
            String date = cursor.getString(cursor.getColumnIndex("date"));
            Map&lt;String, Object&gt; map = new HashMap&lt;String, Object&gt;();
            map.put("tv_content", name);
            map.put("tv_date", date);
            dataList.add(map);
        }
        simple_adapter = new SimpleAdapter(this, dataList, R.layout.item,
                new String[]{"tv_content", "tv_date"}, new int[]{
                R.id.tv_content, R.id.tv_date});
        listview.setAdapter(simple_adapter);
    }



    // 点击listview中某一项的点击监听事件
    @Override
    public void onItemClick(AdapterView&lt;?&gt; arg0, View arg1, int arg2, long arg3) {
        //获取listview中此个item中的内容
        String content = listview.getItemAtPosition(arg2) + "";
        String content1 = content.substring(content.indexOf("=") + 1,
                content.indexOf(","));

        Intent myIntent = new Intent(MainActivity.this, noteEdit.class);
        Bundle bundle = new Bundle();
        bundle.putString("info", content1);
        bundle.putInt("enter_state", 1);
        myIntent.putExtras(bundle);
        startActivity(myIntent);

    }

    // 点击listview中某一项长时间的点击事件
    @Override
    public boolean onItemLongClick(AdapterView&lt;?&gt; arg0, View arg1, final int arg2,
                                   long arg3) {
        Builder builder = new Builder(this);
        builder.setTitle("删除该日志");
        builder.setMessage("确认删除吗？");
        builder.setPositiveButton("确定", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                //获取listview中此个item中的内容
                //删除该行后刷新listview的内容
                String content = listview.getItemAtPosition(arg2) + "";
                String content1 = content.substring(content.indexOf("=") + 1,
                        content.indexOf(","));
                DB.delete("note", "content = ?", new String[]{content1});
                RefreshNotesList();
            }
        });
        builder.setNegativeButton("取消", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
            }
        });
        builder.create();
        builder.show();
        return true;
    }


```

 

 

 

NoteDateBaseHelper:

 

```
import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class NoteDateBaseHelper extends SQLiteOpenHelper {

    public static final String CreateNote = "create table note ("
            + "id integer primary key autoincrement, "
            + "content text , "
            + "date text)";

    public NoteDateBaseHelper(Context context) {
        super(context, "note", null, 1);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CreateNote);
    }

    @Override
    public void onUpgrade(SQLiteDatabase arg0, int arg1, int arg2) {
        // TODO Auto-generated method stub

    }


}
```

 

 

 

 

 

**noteEdit：**

 

```
import android.app.Activity;
import android.content.ContentValues;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import java.text.SimpleDateFormat;
import java.util.Date;

public class noteEdit extends Activity implements OnClickListener {
    private TextView tv_date;
    private EditText et_content;
    private Button btn_ok;
    private Button btn_cancel;
    private NoteDateBaseHelper DBHelper;
    public int enter_state = 0;//用来区分是新建一个note还是更改原来的note
    public String last_content;//用来获取edittext内容

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.edit);

        InitView();
    }

    private void InitView() {
        tv_date = (TextView) findViewById(R.id.tv_date);
        et_content = (EditText) findViewById(R.id.et_content);
        btn_ok = (Button) findViewById(R.id.btn_ok);
        btn_cancel = (Button) findViewById(R.id.btn_cancel);
        DBHelper = new NoteDateBaseHelper(this);

        //获取此时时刻时间
        Date date = new Date();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm");
        String dateString = sdf.format(date);
        tv_date.setText(dateString);

        //接收内容和id
        Bundle myBundle = this.getIntent().getExtras();
        last_content = myBundle.getString("info");
        enter_state = myBundle.getInt("enter_state");
        et_content.setText(last_content);

        btn_cancel.setOnClickListener(this);
        btn_ok.setOnClickListener(this);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.btn_ok:
                SQLiteDatabase db = DBHelper.getReadableDatabase();
                // 获取edittext内容
                String content = et_content.getText().toString();

                // 添加一个新的日志
                if (enter_state == 0) {
                    if (!content.equals("")) {
                        //获取此时时刻时间
                        Date date = new Date();
                        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm");
                        String dateString = sdf.format(date);

                        //向数据库添加信息
                        ContentValues values = new ContentValues();
                        values.put("content", content);
                        values.put("date", dateString);
                        db.insert("note", null, values);
                        finish();
                    } else {
                        Toast.makeText(noteEdit.this, "请输入你的内容！", Toast.LENGTH_SHORT).show();
                    }
                }
                // 查看并修改一个已有的日志
                else {
                    ContentValues values = new ContentValues();
                    values.put("content", content);
                    db.update("note", values, "content = ?", new String[]{last_content});
                    finish();
                }
                break;
            case R.id.btn_cancel:
                finish();
                break;
        }
    }
}

```

 

 

 

**activity_main:**

 

 

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" &gt;

    &lt;TextView
        android:layout_height="wrap_content"
        android:layout_width="fill_parent"
        android:text="记事本"
        android:textStyle="bold"
        android:textSize="22sp"
        android:padding="15dp"
        android:background="#000"
        android:textColor="#fff"
        /&gt;

    &lt;LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="0dp"
        android:layout_weight="1" &gt;

        &lt;ListView
            android:id="@+id/listview"
            android:layout_margin="5dp"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" &gt;
        &lt;/ListView&gt;
    &lt;/LinearLayout&gt;

    &lt;Button
        android:id="@+id/btn_editnote"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="添加备忘录"
        android:padding="10dp"
        android:textSize="20sp" /&gt;

&lt;/LinearLayout&gt;  
```

 

 

 

 

 

**edit:**

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#000"
        android:orientation="vertical"

        android:padding="15dp"&gt;

        &lt;TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="编辑备忘录"
            android:textColor="#fff"
            android:textSize="22sp"
            android:textStyle="bold" /&gt;

        &lt;TextView
            android:id="@+id/tv_date"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:gravity="end"
            android:text="编辑时间"
            android:textColor="#fff" /&gt;
    &lt;/LinearLayout&gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:padding="10dp"
        android:orientation="vertical"&gt;
        &lt;TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="内容编辑："
            android:textColor="#000"
            android:textSize="20sp"
            android:layout_margin="10dp"
            android:textStyle="bold" /&gt;

        &lt;EditText
            android:id="@+id/et_content"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:background="@drawable/edit_text_style"
            android:gravity="start"
            android:hint="此处记录备忘事件"
            android:textSize="20sp" /&gt;

        &lt;LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"&gt;

            &lt;Button
                android:id="@+id/btn_cancel"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="取消" /&gt;

            &lt;Button
                android:id="@+id/btn_ok"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="保存" /&gt;

        &lt;/LinearLayout&gt;
    &lt;/LinearLayout&gt;





&lt;/LinearLayout&gt;  
```

 

 

 

 

 

**item：**

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp"
    android:orientation="vertical"&gt;

    &lt;TextView
        android:id="@+id/tv_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:singleLine="true"
        android:textSize="20sp"
        android:textColor="#000"
        android:text="Large Text" /&gt;

    &lt;TextView
        android:id="@+id/tv_date"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="TextView" /&gt;

&lt;/LinearLayout&gt;  
```

 

 

 

 

 

 

最后附上源码：

 

 
