#android webview拦截加载离线文件
# 概述

客户端经常会通过使用webview来用H5实现一些ios与android双端都有的功能，最常见的比如一些活动页面，内部广告页面等。 这些页面大概会有以下几个特点：
1. 要求webview快一些，白屏时间不要太长1. 要求能够动态更新（因此不能写死在app包中）1. 动态更新不会很频繁
对于这种场景，客户端可以再webview展示之前提前拉取需要展示的内容。（一般会是直接拉取zip包到本地解压，当然也需要有版本判断的逻辑） 然后再webview展示的时候直接根据url拦截，从本地加载文件。 本文主要记录android webview拦截url，加载本地资源的逻辑。

>  
 本文的demo为了精简，本地资源直接是写到了assets文件中。 在实际中使用的时候，应该由客户端自己控制提前下载的时机，下载的地址，以及离线文件的更新逻辑和清理逻辑。 


# demo

实现主要逻辑：
1. 为webview设置自定义的WebViewClient。1. WebViewClient中根据url的host来判断是否有离线文件夹，然后根据".js"，“.html”这些后缀来判断是否需要离线加载的类型，最后根据文件名直接加载。如果加载失败就走网络逻辑。
H5中的主要逻辑：
1. webview会加载一个html，一张png，一个js文件。1. js文件的逻辑是将 “p” 标签中的内容改成 “test string” <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2290.png" width="40%" height="40%">
# 代码

## MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

  private WebView wvMain;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initWebView();
  }

  private void initWebView() {<!-- -->
    wvMain = findViewById(R.id.wv_main);


    wvMain.getSettings().setJavaScriptEnabled(true);
    wvMain.setWebViewClient(new TestWebViewClient(this));
    wvMain.loadUrl("http://xujiajia.blog.csdn.net/index.html");
  }
}

```

## TestWebViewClient.java

```
public class TestWebViewClient extends WebViewClient {<!-- -->
  private static final String TAG = TestWebViewClient.class.getName();

  private Set&lt;String&gt; offlineHosts = new HashSet&lt;&gt;();
  private static final String[] offlineEndString = {<!-- --> ".js", ".html", ".png" };
  private static final String[] mimeTypes =
      {<!-- --> "text/javascript", "text/html", "image/png" };
  private Context mContext;

  public TestWebViewClient(Context context) {<!-- -->
    super();
    mContext = context;

    try {<!-- -->
      String[] hosts = mContext.getAssets().list("");
      if (hosts != null) {<!-- -->
        Collections.addAll(offlineHosts, hosts);
      }
    } catch (Exception e) {<!-- -->
      Log.e(TAG, e.getMessage());
    }
  }

  @Nullable @Override
  public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {<!-- -->
    Uri uri = request.getUrl();
    String host = uri.getHost();
    String fileName = uri.toString().substring(uri.toString().lastIndexOf("/") + 1);

    //判断host命名的文件是否存在
    if (offlineHosts.contains(host)) {<!-- -->
      try {<!-- -->
        //查找具体的文件
        for (int i = 0; i &lt; offlineEndString.length; i++) {<!-- -->
          if (fileName.endsWith(offlineEndString[i])) {<!-- -->

            InputStream is = mContext.getAssets().open(host + "/" + fileName);
            WebResourceResponse response = new WebResourceResponse(mimeTypes[i], "UTF-8", is);
            return response;
          }
        }
      } catch (Exception e) {<!-- -->
        Log.e(TAG, e.getMessage());
      }
    }

    return super.shouldInterceptRequest(view, request);
  }
}


```

## activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    &gt;

  &lt;WebView
      android:id="@+id/wv_main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      /&gt;

&lt;/RelativeLayout&gt;

```

## index.html

```
&lt;html lang="en"&gt;
&lt;body&gt;

&lt;p id="testP"&gt;null&lt;/p&gt;
&lt;img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2291.png" alt="test img"/&gt;

&lt;/body&gt;
&lt;script src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2292.png"&gt;&lt;/script&gt;
&lt;/html&gt;

```

## test.js

```
document.getElementById("testP").innerHTML="test string";

```