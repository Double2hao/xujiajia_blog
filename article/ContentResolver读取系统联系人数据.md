#ContentResolver读取系统联系人数据
由于涉及到个人隐私，此章效果图就不上了吧，layout中只有一个listview用来显示联系人的姓名与电话，读者可以自己尝试运行来查看效果。

 

 

MainActivity:

```
import java.util.ArrayList;
import java.util.List;

import android.os.Bundle;
import android.provider.ContactsContract;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.app.Activity;
import android.database.Cursor;

public class MainActivity extends Activity {

	ListView contactsView;

	ArrayAdapter&lt;String&gt; adapter;

	List&lt;String&gt; contactsList = new ArrayList&lt;String&gt;();

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		contactsView = (ListView) findViewById(R.id.contacts_view);
		adapter = new ArrayAdapter&lt;String&gt;(this, android.R.layout.simple_list_item_1, contactsList);
		contactsView.setAdapter(adapter);
		readContacts();
	}

	//自己使用的时候不要忘记在AndroidManifest.xml中添加访问联系人数据的权限
	private void readContacts() {
		Cursor cursor = null;
		try {
			//查询联系人数据
			//ContactsContract.CommonDataKinds.Phone类已经封装好了URI，NAME,NUMBER等信息
			cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
					null, null, null, null);
			while (cursor.moveToNext()) {
				//获取联系人姓名
				String displayName = cursor.getString(cursor
						.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
				//获取联系人手机号
				String number = cursor.getString(cursor
						.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
				contactsList.add(displayName + "\n" + number);
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (cursor != null) {
				cursor.close();
			}
		}
	}

}
```

 

 

 

 

activity_main.xml:

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" &gt;

    &lt;ListView
        android:id="@+id/contacts_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" &gt;
    &lt;/ListView&gt;

&lt;/LinearLayout&gt;
```

 

 

 

 

 

 

 

 