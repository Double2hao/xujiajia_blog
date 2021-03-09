#android LocationManager的简单使用（获取经纬度信息到textview显示）
此文主要还是写给新手的。

众所周知，基于位置的服务是移动设备的特色，比如现在天气预报可以根据用户所在位置自动选择城市，高德地图的手机导航，发QQ动态的时候我们可以显示自己位置，包括各种APP对百度地图API的使用等等。

那么使用这些都是需要LocationManager了，直接去看那些源码对新手来讲可能有点困难，那就从稍微简单的学起吧。

 

效果图：

<img alt="" class="has" height="700" src="https://img-blog.csdn.net/20151115091233563?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400">

 

 

MainActivity:

 

```
import java.util.List;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;
import org.json.JSONArray;
import org.json.JSONObject;

import android.app.Activity;
import android.content.Context;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends Activity {

    private TextView positionTextView;

    private LocationManager locationManager;

    private String provider;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        positionTextView = (TextView) findViewById(R.id.position_text_view);
        locationManager = (LocationManager) getSystemService(Context.LOCATION_SERVICE);

        // 获取所有可用的位置提供器
        //如果GPS可以用就用GPS，GPS不能用则用网络
        //都不能用的情况下弹出Toast提示用户
        List&lt;String&gt; providerList = locationManager.getProviders(true);
        if (providerList.contains(LocationManager.GPS_PROVIDER)) {
            provider = LocationManager.GPS_PROVIDER;
        } else if (providerList.contains(LocationManager.NETWORK_PROVIDER)) {
            provider = LocationManager.NETWORK_PROVIDER;
        } else {
            Toast.makeText(this, "No location provider to use",
                    Toast.LENGTH_SHORT).show();
            return;
        }

        //使用getLastKnownLocation就可以获取到记录当前位置信息的Location对象了
        //并且用showLocation()显示当前设备的位置信息
        //requestLocationUpdates用于设置位置监听器
        //此处监听器的时间间隔为5秒，距离间隔是5米
        //也就是说每隔5秒或者每移动5米，locationListener中会更新一下位置信息
        Location location = locationManager.getLastKnownLocation(provider);
        if (location != null) {

            showLocation(location);
        }
        locationManager.requestLocationUpdates(provider, 5000, 5,
                locationListener);
    }

    protected void onDestroy() {
        super.onDestroy();
        if (locationManager != null) {
            // 关闭程序时将监听器移除
            locationManager.removeUpdates(locationListener);
        }
    }

    //locationListener中其他3个方法新手不太用得到，笔者在此也不多说了，有兴趣的可以自己去了解一下
    LocationListener locationListener = new LocationListener() {

        @Override
        public void onStatusChanged(String provider, int status, Bundle extras) {
        }

        @Override
        public void onProviderEnabled(String provider) {
        }

        @Override
        public void onProviderDisabled(String provider) {
        }

        @Override
        public void onLocationChanged(Location location) {
            // 更新当前设备的位置信息
            showLocation(location);
        }
    };

    //显示经纬度信息
    private void showLocation(final Location location) {
        String currentPosition = "latitude is " + location.getLatitude() + "\n" + "longitude is "
                + location.getLongitude();
        positionTextView.setText(currentPosition);
    }


}
```

 

 

 activity_main:

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" &gt;

    &lt;TextView
        android:id="@+id/position_text_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20sp"/&gt;

&lt;/LinearLayout&gt;
```

 

 

 

不要忘记在此处加上网络和位置信息的获取权限。

AndroidManifest:

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.locationtest"
    android:versionCode="1"
    android:versionName="1.0" &gt;

    &lt;uses-sdk
        android:minSdkVersion="14"
        android:targetSdkVersion="17" /&gt;

    &lt;uses-permission android:name="android.permission.INTERNET" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" /&gt;

    &lt;application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" &gt;
        &lt;activity
            android:name="com.example.locationtest.MainActivity"
            android:label="@string/app_name" &gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
    &lt;/application&gt;

&lt;/manifest&gt;
```

 

 

 
