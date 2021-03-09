#android JSON解析数据-解析天气预报
#概要 笔者近期做到对天气预报JSON数据解析，在此小记。 天气预报接口： JSON数据如下：

```
{
    "desc": "OK",
    "status": 1000,
    "data": {
        "wendu": "14",
        "ganmao": "天气转凉，空气湿度较大，较易发生感冒，体质较弱的朋友请注意适当防护。",
        "forecast": [
            {
                "fengxiang": "无持续风向",
                "fengli": "微风级",
                "high": "高温 17℃",
                "type": "小雨",
                "low": "低温 10℃",
                "date": "30日星期四"
            },
            {
                "fengxiang": "无持续风向",
                "fengli": "微风级",
                "high": "高温 18℃",
                "type": "多云",
                "low": "低温 7℃",
                "date": "31日星期五"
            },
            {
                "fengxiang": "无持续风向",
                "fengli": "微风级",
                "high": "高温 20℃",
                "type": "晴",
                "low": "低温 8℃",
                "date": "1日星期六"
            },
            {
                "fengxiang": "无持续风向",
                "fengli": "微风级",
                "high": "高温 23℃",
                "type": "晴",
                "low": "低温 10℃",
                "date": "2日星期天"
            },
            {
                "fengxiang": "无持续风向",
                "fengli": "微风级",
                "high": "高温 23℃",
                "type": "多云",
                "low": "低温 12℃",
                "date": "3日星期一"
            }
        ],
        "yesterday": {
            "fl": "微风",
            "fx": "无持续风向",
            "high": "高温 21℃",
            "type": "阴",
            "low": "低温 12℃",
            "date": "29日星期三"
        },
        "aqi": "114",
        "city": "武汉"
    }
}

```

最终解析效果： <img src="https://img-blog.csdn.net/20170330085831844?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述">

#解析概述 1、首先，接到的整个数据可以转化为JSONObject对象。 2、通过整个数据的JSONObject对象获取到data中的数据，也是一个JSONObject对象。在data中就可以获取到此时温度，以及城市等信息。 3、通过data的JSONObject对象可以获取到forecast中的数据，forecast中的数据则是一个JSONArray对象。 4、通过forecast的JSONArray对象可以获取到近几天的天气信息，每一条为一个JSONObject对象。

#代码 方便起见，笔者使用了volley框架，读者新建项目需要在build.gradle的dependencies中添加如下:

```
compile 'eu.the4thfloor.volley:com.android.volley:2015.05.28'

```

MainActivity.java:

```
package com.example.double2.jsontext;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.widget.TextView;
import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.JsonObjectRequest;
import com.android.volley.toolbox.Volley;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

public class MainActivity extends AppCompatActivity {

    private TextView tvMain;
    private RequestQueue mRequestQueue;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        tvMain = (TextView) findViewById(R.id.tv_main);
        mRequestQueue = Volley.newRequestQueue(this);

        JsonObjectRequest mJsonObjectRequest = new JsonObjectRequest(
                "http://wthrcdn.etouch.cn/weather_mini?citykey=101200101",
                null,
                new Response.Listener&lt;JSONObject&gt;() {
                    @Override
                    public void onResponse(JSONObject response) {
                        try {
                            JSONObject data = new JSONObject(response.getString("data"));
                            JSONArray forecast = data.getJSONArray("forecast");
                            JSONObject todayWeather = forecast.getJSONObject(0);

                            String wendu = data.getString("wendu") + "\n";
                            String ganmao = data.getString("ganmao") + "\n";
                            String high = todayWeather.getString("high") + "\n";
                            String low = todayWeather.getString("low") + "\n";
                            String date = todayWeather.getString("date") + "\n";
                            String city = data.getString("city") + "\n";

                            tvMain.setText(wendu + ganmao + high + low + date+city);
                        } catch (JSONException e) {
                            e.printStackTrace();
                        }
                    }
                }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.e("TAG", error.getMessage(), error);
            }
        });

        mRequestQueue.add(mJsonObjectRequest);
    }

}


```

activity_main.xml:

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp"
    &gt;

    &lt;TextView
        android:id="@+id/tv_main"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        /&gt;

&lt;/LinearLayout&gt;


```
