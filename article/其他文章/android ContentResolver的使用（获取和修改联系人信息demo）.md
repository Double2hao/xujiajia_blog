#android ContentResolver的使用（获取和修改联系人信息demo）
ContentProvider和ContentResolver永远是相对的，本章主要是讲ContentResolver的使用，当然是在android系统提供ContentProvider的情况下。

ContentProvider与ContentResolver概念上的东西的就不讲了，主要讲一下ContentResolver的**作用以及使用方法**。

 

**ContentResolver的作用：** 1、可以通过ContentResolver来获取android内部的数据，比如联系人信息、系统的多媒体信息、短信信息等等。

2、可以获取提供了ContentProvider的应用的数据。

 

**ContentResolver的使用方法****：**（参考《疯狂Android讲义第三版》446面）

1、调用Context的getContentResolver()获取ContentResolver对象。

2、根据需要调用ContentResolver的insert(),delete(),updata(),query方法操作数据库。

3、为了操作系统提供的ContentResolver，需要了解该ContentProvider的Uri，以及该ContentProvider所操作的数据列的列名，可以通过查阅Android官方文档来获取这些信息。

4、使用时牢记不要忘记在AndroidManifest中添加权限，并且要及时close掉curcor。

 

**笔者个人意见：**

1、建议新手读者在学完SqLite之后再来学习ContentResolver的内容，ContentResolver的insert(),delete(),updata(),query四个操作数据库的方法均与SQLite中的相同。

2、ContentProvider的Uri，以及该ContentProvider所操作的数据列的列名都不需要死记硬背，需要用的时候查阅一下即可，新手此处最好能自己全部写一下，对ContentResolver的使用有一个总体的了解。

 

笔者demo效果：**（源码在文章结尾）**

<img alt="" class="has" height="450" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039201270.png" width="300">

 

代码截图：

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039203241.png">

 

在demo中用到的ContentProvider的两个Uri如下：

 

**管理联系人的Uri。**

 

 

ContactsContract.Contacts.CONTENT_URI

 

**管理联系人电话的Uri。**

ContactsContract.CommonDataKinds.Phone.CONTENT_URI

 

 

AndroidManifest：

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.double2.mcontentprovider"&gt;

    &lt;application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"&gt;
        &lt;activity android:name=".MainActivity"&gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
        &lt;activity android:name=".ShowActivity"/&gt;
    &lt;/application&gt;

    &lt;uses-permission android:name="android.permission.READ_CONTACTS"/&gt;
    &lt;uses-permission android:name="android.permission.WRITE_CONTACTS"/&gt;
&lt;/manifest&gt;
```

 

 

 

 

 

 

 

 

MainActivity:

 

```
package com.example.double2.mcontentprovider;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.ContentUris;
import android.content.ContentValues;
import android.content.DialogInterface;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.provider.ContactsContract;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

public class MainActivity extends Activity {

    private Button btnShow;
    private Button btnAdd;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btnShow=(Button)findViewById(R.id.btn_show);
        //点击后跳转到ShowActivity
        btnShow.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ToShowActivity();
            }
        });
        btnAdd=(Button)findViewById(R.id.btn_add);
        //点击后跳出添加信息的dialog
        btnAdd.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showAddDialog();
            }
        });
    }

    private void showAddDialog() {
        View dialog_add=getLayoutInflater().inflate(R.layout.dialog_add, null);
        final EditText etName=(EditText)dialog_add.findViewById(R.id.et_name);
        final EditText etPhone=(EditText)dialog_add.findViewById(R.id.et_phone);

        //初始化dialog
        AlertDialog.Builder dialog=new AlertDialog.Builder(MainActivity.this);
        dialog.setTitle("添加联系人")
                .setView(dialog_add)
                .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        String mName=etName.getText().toString();
                        String mPhone=etPhone.getText().toString();

                        if(mName.equals("")||mPhone.equals(""))
                            Toast.makeText(MainActivity.this, "联系人姓名或电话不能为空", Toast.LENGTH_SHORT).show();
                        else
                        addMyContact(mName,mPhone);
                    }
                })
                .setNegativeButton("取消",null)
                .create()
                .show();
    }

    private void addMyContact(String mName, String mPhone) {
        //创建一个空的ContentValues
        ContentValues values=new ContentValues();
        //向ContactsContract.RawContacts.CONTENT_URI执行一个空值插入
        //目的是获取系统返货的rawContactId，以便添加联系人名字和电话使用同一个id
        Uri rawContactUri=getContentResolver().insert(
                ContactsContract.RawContacts.CONTENT_URI,values);
        long rawContactId= ContentUris.parseId(rawContactUri);

        //清空values
        //设置id
        //设置内容类型
        //设置联系人姓名
        values.clear();
        values.put(ContactsContract.Data.RAW_CONTACT_ID, rawContactId);
        values.put(ContactsContract.Data.MIMETYPE,
                ContactsContract.CommonDataKinds.StructuredName.CONTENT_ITEM_TYPE);
        values.put(ContactsContract.CommonDataKinds.StructuredName.GIVEN_NAME, mName);
        //向联系人URI添加联系人姓名
        getContentResolver().insert(ContactsContract.Data.CONTENT_URI,values);

        //清空values
        //设置id
        //设置内容类型
        //设置联系人电话
        //设置电话类型
        values.clear();
        values.put(ContactsContract.Data.RAW_CONTACT_ID, rawContactId);
        values.put(ContactsContract.Data.MIMETYPE, ContactsContract.CommonDataKinds.Phone.CONTENT_ITEM_TYPE);
        values.put(ContactsContract.CommonDataKinds.Phone.NUMBER,mPhone);
        values.put(ContactsContract.CommonDataKinds.Phone.TYPE, ContactsContract.CommonDataKinds.Phone.TYPE_MOBILE);
        getContentResolver().insert(ContactsContract.Data.CONTENT_URI, values);

        //使用toast提示用户信息添加成功
        Toast.makeText(MainActivity.this, "联系人数据添加成功！", Toast.LENGTH_SHORT).show();
    }

    private void ToShowActivity() {
        Intent intent=new Intent(MainActivity.this,ShowActivity.class);
        startActivity(intent);
    }

}

```

 

 

 

 

 

ShowActivity:

 

```
package com.example.double2.mcontentprovider;

import android.app.Activity;
import android.database.Cursor;
import android.os.Bundle;
import android.provider.ContactsContract;
import android.util.Log;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.SimpleAdapter;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Created by Double2号 on 2016/4/6.
 */
public class ShowActivity extends Activity {

    ListView lvContacts;
    List&lt;Map&lt;String,String&gt;&gt; mListItems;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_show);

        lvContacts=(ListView)findViewById(R.id.lv_contacts);

        //此处主要思路是获取联系人的姓名和电话存入mListItems
        //然后通过mListItems创建SimpleAdapter
        mListItems=new ArrayList&lt;Map&lt;String,String&gt;&gt;();
        getContactsData();
        SimpleAdapter mSimpleAdapter=new SimpleAdapter(this,mListItems,R.layout.item_list,
                new String[]{"name","phone"},
                new int[]{R.id.tv_item_name,R.id.tv_item_phone});
        lvContacts.setAdapter(mSimpleAdapter);
    }

    public void getContactsData() {
        //使用ContentResolver查找联系人数据
        Cursor cursor=getContentResolver().query(
                ContactsContract.Contacts.CONTENT_URI,null,null, null,null);
        //便利查询结果，获取系统中的所有人
        while (cursor.moveToNext()){
            //获取联系人id
            String contactId=cursor.getString(cursor.getColumnIndex(
                    ContactsContract.Contacts._ID));
            Log.i("xujiajia",contactId);
            //获取联系人姓名
            String name=cursor.getString(cursor.getColumnIndex(
                    ContactsContract.Contacts.DISPLAY_NAME));
            Log.i("xujiajia", name);

            //使用过ContentResolver通过id查找联系人的电话
            //此处为了方便显示，只取联系人的第一个号码（可能有多个号码）
            Cursor phones=getContentResolver().query(
                    ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                    null, ContactsContract.CommonDataKinds.Phone.CONTACT_ID + "=" + contactId
                    , null, null);
            phones.moveToNext();
            String phone=phones.getString(phones.getColumnIndex(
                    ContactsContract.CommonDataKinds.Phone.NUMBER));
            Log.i("xujiajia", phone);
            //使用完毕关闭Cursor
            phones.close();

            //创建Map添加到mListItems中用于创建SimpleAdapter
            Map&lt;String,String&gt; listItem=new HashMap&lt;String,String&gt;();
            listItem.put("name",name);
            listItem.put("phone",phone);
            mListItems.add(listItem);
        }
        //使用完毕关闭Cursor
        cursor.close();
    }
}

```

 

 

activity_main:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:orientation="vertical"
    &gt;

    &lt;Button
        android:id="@+id/btn_show"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="查看联系人"
        /&gt;

    &lt;Button
        android:id="@+id/btn_add"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="添加联系人" /&gt;
&lt;/LinearLayout&gt;

```

 activity_show:

 

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"&gt;

    &lt;ListView
        android:id="@+id/lv_contacts"
        android:layout_width="match_parent"
        android:layout_height="match_parent" /&gt;
&lt;/LinearLayout&gt;
```

 

 

 

 

 

 

 

dialog_add:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp"&gt;

    &lt;TableRow&gt;

        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="联系人姓名：" /&gt;

        &lt;EditText
            android:id="@+id/et_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1" /&gt;
    &lt;/TableRow&gt;

    &lt;TableRow&gt;

        &lt;TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="联系人电话：" /&gt;

        &lt;EditText
            android:id="@+id/et_phone"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1" /&gt;
    &lt;/TableRow&gt;
&lt;/TableLayout&gt;
```

 item_list:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp"
    android:orientation="vertical"&gt;

    &lt;TextView
        android:id="@+id/tv_item_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        tools:text="联系人姓名"
        android:textSize="20sp"/&gt;
    &lt;TextView
        android:id="@+id/tv_item_phone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        tools:text="联系人电话"
        android:textSize="20sp"/&gt;
&lt;/LinearLayout&gt;
```

 

 

 

 

源码地址：

 

 

 

 

 