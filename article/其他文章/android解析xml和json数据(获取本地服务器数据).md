#android解析xml和json数据(获取本地服务器数据)
前提概要

此知识点可能比较抽象，难以理解，笔者在学到这里的时候也是一度放弃，然后又一度鼓励自己。（其实去年已经接触到了这边的知识，由于过于害怕，到现在才开始专心学，懂了一点）也是希望各位读者给自己一点耐心，专心花几个小时学一下，还是能懂一点的。另外，笔者的博文也仅带表个人意见，还有很多讲的不好的地方，还是希望大家能够指出。新手的话还是建议拿本参考书好好琢磨下。此篇博文也仅仅做入门用，分析HTTP协议可能需要花费一整本书的篇幅。笔者的想法，还是先入门，之后用多少，学多少。

 

首先还是看一下效果

 

**服务器上的XML文件：**

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/240.png">

 

**解析XML后通过Log输出：**

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/241.png">

 

**服务器上的json文件：**

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/242.png">

 

**解析json后通过Log输出：**

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/243.png">

 

 

接下来讲一下实际操作吧：（准备工作可能比较繁琐，但是很简单）

1、下载apache服务器安装包，笔者下载的是2.4。至于具体情况还是请读者百度了，并不困难。在此稍微提一下，安装apache的目录中不要携带中文字符，不然会有bug。（这点上android studio也是如此，在此也是建议做开发的各位最好习惯用英文命名文件夹）

 

2、在apache的，htdocs目录下新建一个xml文件与一个json文件用于测试。笔者的文件路径是D:\apache\Apache24\htdocs，如图：

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/244.png">

 

并且在两个文件中写好相应的测试数据：

 

get_data.xml

 

&lt;apps&gt; &lt;app&gt; &lt;id&gt;1&lt;/id&gt; &lt;name&gt;Google Maps&lt;/name&gt; &lt;version&gt;1.0&lt;/version&gt; &lt;/app&gt; &lt;app&gt; &lt;id&gt;2&lt;/id&gt; &lt;name&gt;Chrome&lt;/name&gt; &lt;version&gt;2.1&lt;/version&gt; &lt;/app&gt; &lt;app&gt; &lt;id&gt;3&lt;/id&gt; &lt;name&gt;Google Maps&lt;/name&gt; &lt;version&gt;2.3&lt;/version&gt; &lt;/app&gt; &lt;/apps&gt;

 

get_data.json

 

[{"id":"5","version":"5.5","name":"Angry Birds"}, {"id":"6","version":"7.0","name":"Clash of Clans"}, {"id":"7","version":"3.5","name":"Hey Day"}]

 

如图：

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/245.png">

 

写好之后在网址中输入 127.0.0.1/get_data.xml就可以显示出来了，如图：

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/246.png">

 

 

3、最后就是对代码的理解了，笔者的代码中包含**XML格式的pull解析方式**和**JSON格式的JSONObject解析方式。**但是实际上XML还包括SAX解析方式，JSON还包括GSON（此方式很简单）、jackson等。新手读者可能看到如此多不懂的东西比较畏惧——但是再次强调，**不要害怕，会了之后也就这么回事**，花几个小时静静地学一下，不难理解的。（也是推荐新手找一本参考书哦）

 

 

直接开启一个项目，在MainActivity中复制如下代码，然后在AndroidManifest中设置访问网络的权限就可以用了。使用虚拟机就可以看到log打印出的内容了。（此处一定要用虚拟机，手机可无法识别本地服务器）

MainActivity:

 

```
import java.io.StringReader;


import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;
import org.json.JSONArray;
import org.json.JSONObject;
import org.xmlpull.v1.XmlPullParser;
import org.xmlpull.v1.XmlPullParserFactory;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		//测试
		sendRequestWithHttpClient();

	}


	private void sendRequestWithHttpClient() {
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					HttpClient httpClient = new DefaultHttpClient();
					//10.0.2.2对于模拟器来说就是电脑本机的IP地址
					//指定访问的服务器地址是电脑本机
//					HttpGet httpGet = new HttpGet("http://10.0.2.2/get_data.xml");
					HttpGet httpGet = new HttpGet("http://10.0.2.2/get_data.json");
					HttpResponse httpResponse = httpClient.execute(httpGet);
					if (httpResponse.getStatusLine().getStatusCode() == 200) {
						// 请求和响应都成功了
						HttpEntity entity = httpResponse.getEntity();
						String response = EntityUtils.toString(entity, "utf-8");

						//解析xml格式数据
//						parseXMLWithPull(response);

						//解析JSON格式数据
						parseJSONWithJSONObject(response);


					}
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}).start();
	}


	private void parseJSONWithJSONObject(String jsonData) {
		try {
			//在服务器中定义的是一个JSON数据
			//因此在这里首先是将服务器返回的诗句传到一个JSONArray对象中
			//然后循环遍历这个JSONArray，从中去除每一个元素都是JSONbject对象
			//JSONbject对象中又会包含id、name、和version这些数据
			//最后调用getString方法将这些数据取出
			JSONArray jsonArray = new JSONArray(jsonData);
			for (int i = 0; i &lt; jsonArray.length(); i++) {
				JSONObject jsonObject = jsonArray.getJSONObject(i);
				String id = jsonObject.getString("id");
				String name = jsonObject.getString("name");
				String version = jsonObject.getString("version");
				Log.d("MainActivity", "id is " + id);
				Log.d("MainActivity", "name is " + name);
				Log.d("MainActivity", "version is " + version);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}


	private void parseXMLWithPull(String xmlData) {
		try {
			//首先获取一个XmlPullParserFactory实例
			//借助这个实例得到XmlPullParser对象
			//调用setInput（）方法将服务器返回的XML数据设置进去
			//通过getEventType得到该解析事件
			XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
			XmlPullParser xmlPullParser = factory.newPullParser();
			xmlPullParser.setInput(new StringReader(xmlData));
			int eventType = xmlPullParser.getEventType();
			String id = "";
			String name = "";
			String version = "";
			while (eventType != XmlPullParser.END_DOCUMENT) {
				//通过getName（）得到当前结点的名字
				//如果发现id、name、version就调用nextText（）方法来获取结点内具体内容
				String nodeName = xmlPullParser.getName();
				switch (eventType) {
				// 开始解析某个结点
				case XmlPullParser.START_TAG: {
					if ("id".equals(nodeName)) {
						id = xmlPullParser.nextText();
					} else if ("name".equals(nodeName)) {
						name = xmlPullParser.nextText();
					} else if ("version".equals(nodeName)) {
						version = xmlPullParser.nextText();
					}
					break;
				}
				// 完成解析某个结点
				case XmlPullParser.END_TAG: {
					if ("app".equals(nodeName)) {
						Log.d("MainActivity", "id is " + id);
						Log.d("MainActivity", "name is " + name);
						Log.d("MainActivity", "version is " + version);
					}
					break;
				}
				default:
					break;
				}
				eventType = xmlPullParser.next();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

 

 

 

 

 

AndroidManifest:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.networktest"
    android:versionCode="1"
    android:versionName="1.0" &gt;

    &lt;uses-sdk
        android:minSdkVersion="14"
        android:targetSdkVersion="17" /&gt;

    &lt;uses-permission android:name="android.permission.INTERNET" /&gt;

    &lt;application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" &gt;
        &lt;activity
            android:name="com.example.networktest.MainActivity"
            android:label="@string/app_name" &gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
    &lt;/application&gt;

&lt;/manifest&gt;
```

 

 

 

 

 

 