#webview显示html代码（可用于新闻浏览界面）
 

近期在制作校园app的新闻界面，自然也要设计到新闻浏览的一个问题，还是先展示一下效果：

 

<img alt="" class="has" height="1100" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911859260.png " width="500">

 

使用WebView来做新闻的显示特别简单，只需要从该网站获取到新闻的html代码用String存储，随后使用WebView显示便可以了。

 

当然，对于要求比较高的同学来讲，还是可以通过部分获取内容的方式来实现此界面的。但是此方法涉及了较多的数据处理以及xml设计等，本人在审美方面实在有点捉急，于是便使用这个讨巧的方法了。

 

MainActivity:

 

```
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.webkit.WebView;

public class MainActivity extends AppCompatActivity {

    private WebView test;
    private String blank = "        ";
    private String notice1 =
            "&lt;div id=\"innerPadding\"&gt;\n" +
                    "                \n" +
                    "\t\t\t\t&lt;div class=\"artTitle\"&gt;\n" +
                    "\t\t\t\t\t&lt;h2&gt;尼泊尔政府代表团来我校考察数字化校园建设&lt;/h2&gt;\n" +
                    "\t\t\t\t\t&lt;span&gt;( 点击次数：46次   发布时间：2015-09-23   \n" +
                    "\t\t\t\t\t 作者：杨鲜 见习记者郑孟娜    来源：本站)&lt;/span&gt;\n" +
                    "\t\t\t\t&lt;/div&gt;\n" +
                    "\n" +
                    "\t\t\t\t&lt;div class=\"artContent\"&gt;\n" +
                    "\t\t\t\t\t&lt;div id=\"detailNews\"&gt;&lt;p&gt;\n" +
                    "\t&lt;span style=\"line-height:1.5;\"&gt;    新闻网讯（记者&lt;/span&gt;&lt;span style=\"line-height:1.5;\"&gt;杨鲜 见习记者郑孟娜）9月21日，尼泊尔政府信息与通讯管理部官员访问团一行19人来我校考察数字化校园建设。&lt;/span&gt;&lt;span style=\"line-height:1.5;\"&gt;在黄家湖校区图书馆二楼，我校现代教育信息中心主任万维新向外宾介绍了我校悠久的办学历史并详细介绍了我校数字化校园建设情况。&lt;/span&gt;\n" +
                    "&lt;/p&gt;\n" +
                    "&lt;p&gt;\n" +
                    "\t&lt;span style=\"line-height:1.5;\"&gt;&lt;br&gt;\n" +
                    "&lt;/span&gt;\n" +
                    "&lt;/p&gt;\n" +
                    "&lt;p&gt;\n" +
                    "\t    据了解，我校校园网自1998年开始建设，1999年投入运行，经过不断地升级改造，已成为安全可靠、服务种类丰富的现代化网络中心。\n" +
                    "&lt;/p&gt;\n" +
                    "&lt;p&gt;\n" +
                    "\t&lt;br&gt;\n" +
                    "&lt;/p&gt;\n" +
                    "&lt;p&gt;\n" +
                    "\t    万维新介绍说，我校已经建立了一套完备的管理制度，为校内办公、教学、科研提供了稳定的网络环境。2014年我校与企业合作，在湖北省率先实现了全校园无线网覆盖。在未来几年内，我们将以建设数字化校园为中心，进一步扩大校园网络的规模，拓展应用范围，把学校建设成真正的智慧校园。&lt;span style=\"line-height:1.5;\"&gt;锐捷网络公司技术人员就外宾提出的各种技术问题一一做了解答。&lt;/span&gt;\n" +
                    "&lt;/p&gt;\n" +
                    "&lt;p&gt;\n" +
                    "\t \n" +
                    "&lt;/p&gt;\n" +
                    "&lt;p&gt;\n" +
                    "\t    尼泊尔代表团负责人PARASHURAM司长对我校优美的环境赞叹不已，他代表访问团感谢我校给他们提供了一次学习的机会，并希望今后能多与我校交流，创建更完善的数字信息系统。\n" +
                    "&lt;/p&gt;\n" +
                    "&lt;p&gt;\n" +
                    "\t&lt;br&gt;\n" +
                    "&lt;/p&gt;&lt;/div&gt;\n" +
                    "\t\t\t\t\t&lt;div id=\"artEnd\"&gt;\n" +
                    "\t\t\t\t\t\t&lt;span class=\"artKey\"&gt;文章关键字: &lt;/span&gt;\n" +
                    "\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=校园\"&gt;校园&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=代表团\"&gt;代表团&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=政府\"&gt;政府&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=考察\"&gt;考察&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=考\"&gt;考&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=数字\"&gt;数字&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=来\"&gt;来&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=建设\"&gt;建设&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=我校\"&gt;我校&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=我\"&gt;我&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\t   &lt;a href=\"/searchNews.action?key=代表\"&gt;代表&lt;/a&gt;\n" +
                    "\t\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t\n" +
                    "\t\t\t\t\t\t&lt;span id=\"editor\"&gt;(编辑：程毓)&lt;/span&gt;\n" +
                    "\t\t\t\t\t&lt;/div&gt;\n" +
                    "\t\t\t\t&lt;/div&gt;\n" +
                    "\t\t\t\t&lt;!-- 分享 --&gt;\n" +
                    "\t\t\t\t&lt;div class=\"bshare-custom\"&gt;\n" ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        test = (WebView) findViewById(R.id.test);
        test.loadDataWithBaseURL("about:blank", notice1, "text/html", "utf-8", null);


    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
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
    tools:context=".MainActivity"&gt;


    &lt;WebView
        android:id="@+id/test"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"&gt;&lt;/WebView&gt;

&lt;/RelativeLayout&gt;
```

 

 

 此方法也主要是webview对html的显示，至于html代码的获取可以参考我的另一篇博客：

 

 

 

 

本人博客，android均为新手，闻过则喜，望各位前辈不吝指点批评。

 