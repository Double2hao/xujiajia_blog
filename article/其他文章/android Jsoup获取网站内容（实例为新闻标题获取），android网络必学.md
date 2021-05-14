#android Jsoup获取网站内容（实例为新闻标题获取），android网络必学
近期做简单的新闻客户端界面使用到了Jsoup获取，使用起来特别方便，这也是被我一个学长称为学android网络必学的一个东西，在此也是分享一下自己近期所学。

 

首先还是给出效果：

<img alt="" class="has" height="1000" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911038950.png " width="500">

 

 

上面是通过textview显示的一个从网站上获取的所有内容的显示，下面是通过listview显示一下获取的新闻的标题，如此显示比较便于理解。

 

 

MainActivity：

 

```
import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.AsyncTask;
import android.os.Bundle;
import android.text.method.ScrollingMovementMethod;
import android.util.Log;
import android.view.Menu;
import android.view.View;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.TextView;

import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import java.util.ArrayList;
import java.util.List;

@SuppressWarnings("unused")
public class MainActivity extends Activity {
    private TextView TV_HTMLCode;
    //此处搞一个TextView主要来显示News列表里面存储的内容，仅仅便于分析和理解

    private String URL_EOL = "http://www.cnwust.com/newsList/1_1",
            TAG = "ATAG";
    //这是索要获取内容的网址

    private List&lt;News&gt; NewsList;
    //自定义的News的类，用于存放索要获取新闻的目录、时间以及点击后显示的网址

    private ListView LV_Result;
    private ArrayAdapter&lt;String&gt; LV_Adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        LV_Result = (ListView) findViewById(R.id.LV_Result);
        TV_HTMLCode = (TextView) findViewById(R.id.TV_HTMLCode);
        TV_HTMLCode.setMovementMethod(ScrollingMovementMethod.getInstance());

        ConnectTask C1 = new ConnectTask();
        C1.execute();

    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    public class ConnectTask extends AsyncTask&lt;Void, Void, String&gt; {

        @Override
        protected String doInBackground(Void... params) {
            String result = ConnectEOL();
            return result;
        }

        @Override
        protected void onPostExecute(String result) {
            // TV_HTMLCode.setText(result);
            NewsList = getNews(result);
            List&lt;String&gt; NewsTitles = new ArrayList&lt;String&gt;();
            for (News news : NewsList) {
                TV_HTMLCode.append(news.getNewsTitle() + "\n");
                TV_HTMLCode.append(news.getNewsTime() + "\n");
                TV_HTMLCode.append(news.getNewsUrl() + "\n");
                NewsTitles.add(news.getNewsTitle());
            }
        /* 为ListView添加适配器 */

            LV_Adapter = new ArrayAdapter&lt;String&gt;(MainActivity.this,
                    android.R.layout.simple_list_item_1, NewsTitles);
            LV_Result.setAdapter(LV_Adapter);

        /* 为ListView添加点击打开对应网页功能 */
            LV_Result.setOnItemClickListener(new OnItemClickListener() {

                @Override
                public void onItemClick(AdapterView&lt;?&gt; arg0, View arg1,
                                        int arg2, long arg3) {
                    final Uri uri = Uri.parse(NewsList.get(arg2).getNewsUrl());
                    final Intent it = new Intent(Intent.ACTION_VIEW, uri);
                    startActivity(it);
                }

            });
            //此处为了方便就点击就直接调用设备默认浏览器打开网址

            super.onPostExecute(result);


        }

    }

    /* 连接EOL的方法 返回整个网页经过截取之后的的源代码 */
    public String ConnectEOL() {
        String result = "";
        try {
            HttpClient httpclient = new DefaultHttpClient();
            HttpPost httppost = new HttpPost(URL_EOL);
            HttpResponse response = httpclient.execute(httppost);
            String Res = EntityUtils.toString(response.getEntity(), "UTF-8");

            int st = Res.indexOf("&lt;div id=\"result\"&gt;");
            int ed = Res.indexOf("&lt;div id=\"pager\"&gt;");
            //这边算是最重要的部分，代码获取的便是这两段之间的部分。

            String content = Res.substring(st, ed);
            st = content.indexOf("&lt;ul&gt;") + 4;
            ed = content.indexOf("&lt;/ul&gt;");
            content = content.substring(st, ed);
            result = content;
        } catch (Exception e) {
            Log.d(TAG, e.toString());
        }
        return result;
    }

    /* 对源代码进行解析截取的方法 返回一个News数组 */
    public List&lt;News&gt; getNews(String HTMLCode) {
        List&lt;News&gt; newsList = new ArrayList&lt;News&gt;();
        Document doc = Jsoup.parse(HTMLCode);
        Log.d(TAG, "解析html中");
        Elements lis = doc.getElementsByTag("li");
        Log.d(TAG, "lis的size " + lis.size());
        for (Element li : lis) {
            String newstime = li.getElementsByTag("span").text();
            String newstitle = li.getElementsByTag("a").text();
            String newsurl = li.getElementsByTag("a").attr("href");
            //这三段算是Jsoup从html中获取内容的关键了，很容易理解。

            newsurl = newsurl.replace("/news", "http://www.cnwust.com/news");
            //直接从html的代码中获取的URL是相对路径，此处使用replace改为绝对路径

            Log.d(TAG, newstime);
            Log.d(TAG, newstitle);
            Log.d(TAG, newsurl);

            News newst = new News();
            newst.setNewsTime(newstime);
            newst.setNewsTitle(newstitle);
            newst.setNewsUrl(newsurl);
            newsList.add(newst);
        }
        return newsList;
    }
}
```

 

 

 

 

News：

 

```
public class News {
    private String newsTime;
    private String newsUrl;
    private String newsTitle;

    public News() {

    }

    public News(String newsTitle, String newsTime, String newsUrl) {
        this.newsTime = newsTime;
        this.newsUrl = newsUrl;
        this.newsTitle = newsTitle;
    }

    public String getNewsTime() {
        return newsTime;
    }

    public void setNewsTime(String newsTime) {
        this.newsTime = newsTime;
    }

    public String getNewsUrl() {
        return newsUrl;
    }

    public void setNewsUrl(String newsUrl) {
        this.newsUrl = newsUrl;
    }

    public String getNewsTitle() {
        return newsTitle;
    }

    public void setNewsTitle(String newsTitle) {
        this.newsTitle = newsTitle;
    }

}

```

 

 

activity_main:

 

```
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".NewsList" &gt;

    &lt;TextView
        android:id="@+id/TV_HTMLCode"
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:layout_above="@+id/LV_Result"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"
        android:scrollbars="vertical" /&gt;

    &lt;ListView
        android:id="@+id/LV_Result"
        android:layout_width="match_parent"
        android:layout_height="230dp"
        android:layout_alignLeft="@+id/TV_HTMLCode"
        android:layout_alignParentBottom="true" &gt;
    &lt;/ListView&gt;

&lt;/RelativeLayout&gt;

```

 

此处对html代码的解析可能部分新手还是不太清楚，在此也是建议使用chrome浏览器，可以直接查看网站的源码。（有部分加密的网站看不到）下面看一下具体使用的截图：

 

 

 

1、首先先要打开到你要获取内容的网站

 

<img alt="" class="has" height="400" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911041951.png " width="700">

 

 

2、右击你要获取的内容，并选择  审查元素。

 

<img alt="" class="has" height="400" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911043682.png " width="700">

 

3、使用Jsoup解析html代码。

 

<img alt="" class="has" height="400" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911046023.png " width="700">

 

 

最后是附上源码下载地址：

 

 

本人博客，android均为新手，闻过则喜，望各位前辈不吝指点批评。  