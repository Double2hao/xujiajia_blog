#使用百度地图API，定位并显示自己的位置
使用百度地图的API其实在代码上并不是特别困难，参考一下官网开发指南或者技术书籍都比较好理解，主要的比较麻烦的地方是出在jar，so文件的导入与的使用，本篇主要是给一个代码的参考，有读者在其他方面遇到困难的可以参考笔者的其他几篇博客。

 

jar与so：  

 





 

# 

 

 

 

 

# 

 

 

 

# 

 



 

：





 

 

还是先上一下效果：

 

<img alt="" class="has" height="700" src="https://img-blog.csdn.net/20151117213124387?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400">

 

 

MainActivity:

 

```
import android.app.Activity;
import android.content.Context;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.os.Bundle;
import android.widget.Toast;


import com.baidu.mapapi.SDKInitializer;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.MapStatusUpdate;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.map.MyLocationConfiguration;
import com.baidu.mapapi.map.MyLocationData;
import com.baidu.mapapi.model.LatLng;

import java.util.List;

public class MainActivity extends Activity {

    private MapView mapView;
    private BaiduMap baiduMap;
    private LocationManager locationManager;
    private  String provider;
    private boolean isFirstLocate =true;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SDKInitializer.initialize(getApplicationContext());
        setContentView(R.layout.activity_main);
        mapView = (MapView) findViewById(R.id.map_view);
        baiduMap=mapView.getMap();
        //设置位置提供器
        setLovationManager();
        //将显示位置的功能开启
        baiduMap.setMyLocationEnabled(true);
    }

    private void setLovationManager() {
        locationManager=(LocationManager)getSystemService(Context.LOCATION_SERVICE);
        //获取所有可用的位置提供器
        List&lt;String&gt; providerList=locationManager.getProviders(true);
        if(providerList.contains(LocationManager.GPS_PROVIDER)){
            provider=LocationManager.GPS_PROVIDER;
        }else if(providerList.contains(LocationManager.NETWORK_PROVIDER)){
            provider=LocationManager.NETWORK_PROVIDER;
        }else {
            //当前没有可用的位置提供器时，弹出Toast提示
            Toast.makeText(this,"没有可用的位置提供器",Toast.LENGTH_SHORT).show();
            return;
        }
        Location location=locationManager.getLastKnownLocation(provider);
        if(location!=null){
            navigateTo(location);
        }

        locationManager.requestLocationUpdates(provider,5000,5,locationListener);
    }

    private void navigateTo(Location location) {
        //如果是第一次创建，就获取位置信息并且将地图移到当前位置
        //为防止地图被反复移动，所以就只在第一次创建时执行
        if(isFirstLocate){
            //LatLng对象主要用来存放经纬度
            //zoomTo是用来设置百度地图的缩放级别，范围为3~19，数值越大越精确
            LatLng ll=new LatLng(location.getLatitude(),location.getLongitude());
            MapStatusUpdate update= MapStatusUpdateFactory.newLatLng(ll);
            baiduMap.animateMapStatus(update);
            update=MapStatusUpdateFactory.zoomTo(16f);
            baiduMap.animateMapStatus(update);
            isFirstLocate=false;
        }

        //封装设备当前位置并且显示在地图上
        //由于设备在地图上显示的位置会根据我们当前位置而改变，所以写到if外面
        MyLocationData.Builder locationBuilder=new MyLocationData.Builder();
        locationBuilder.latitude(location.getLatitude());
        locationBuilder.longitude(location.getLongitude());
        MyLocationData locationData=locationBuilder.build();
        baiduMap.setMyLocationData(locationData);
    }

    LocationListener locationListener =new LocationListener() {
        @Override
        public void onLocationChanged(Location location) {
            if(locationManager!=null)
                navigateTo(location);
        }

        @Override
        public void onStatusChanged(String s, int i, Bundle bundle) {

        }

        @Override
        public void onProviderEnabled(String s) {

        }

        @Override
        public void onProviderDisabled(String s) {

        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //最后要销毁mapView
        //关闭程序时将监听器移除
        //关闭可以显示位置的功能
        mapView.onDestroy();
        if(locationManager!=null){
            locationManager.removeUpdates(locationListener);
        }
        baiduMap.setMyLocationEnabled(false);
    }

    @Override
    protected void onPause() {
        super.onPause();
        mapView.onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();
        mapView.onResume();
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
    &gt;
    &lt;com.baidu.mapapi.map.MapView
        android:id="@+id/map_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:clickable="true"&gt;

    &lt;/com.baidu.mapapi.map.MapView&gt;

&lt;/LinearLayout&gt;
```

 

 

 

 

 

Androidmanifest:

 

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.baidumaptest2"&gt;

    &lt;application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"&gt;
        &lt;meta-data
            android:name="com.baidu.lbsapi.API_KEY"
            android:value="lPdICf3mOjPpwXsUzv8Omgec"/&gt;
        &lt;activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:theme="@style/AppTheme.NoActionBar"&gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;

                &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
    &lt;/application&gt;

    //以下为权限设置
    &lt;uses-permission android:name="android.permission.GET_ACCOUNTS" /&gt;
    &lt;uses-permission android:name="android.permission.USE_CREDENTIALS" /&gt;
    &lt;uses-permission android:name="android.permission.MANAGE_ACCOUNTS" /&gt;
    &lt;uses-permission android:name="android.permission.AUTHENTICATE_ACCOUNTS" /&gt;
    &lt;permission android:name="android.permission.BAIDU_LOCATION_SERVICE" &gt;
    &lt;/permission&gt;
    &lt;uses-permission android:name="android.permission.BAIDU_LOCATION_SERVICE" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.INTERNET" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.ACCESS_MOCK_LOCATION" &gt;
    &lt;/uses-permission&gt;
    &lt;!-- &lt;uses-permission android:name="android.permission.WRITE_APN_SETTINGS"&gt;&lt;/uses-permission&gt; --&gt;
    &lt;uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="com.android.launcher.permission.READ_SETTINGS" /&gt;
    &lt;uses-permission android:name="android.permission.WAKE_LOCK" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.CHANGE_WIFI_STATE" /&gt;
    &lt;uses-permission android:name="android.permission.ACCESS_WIFI_STATE" /&gt;
    &lt;uses-permission android:name="android.permission.ACCESS_GPS" /&gt;
    &lt;uses-permission android:name="android.permission.READ_PHONE_STATE" /&gt;
    &lt;uses-permission android:name="android.permission.READ_CONTACTS" /&gt;
    &lt;uses-permission android:name="android.permission.CALL_PHONE" /&gt;
    &lt;uses-permission android:name="android.permission.READ_SMS" /&gt;
    &lt;uses-permission android:name="android.permission.SEND_SMS" /&gt;
    &lt;!-- SDK1.5需要android.permission.GET_TASKS权限判断本程序是否为当前运行的应用? --&gt;
    &lt;uses-permission android:name="android.permission.GET_TASKS" /&gt;
    &lt;uses-permission android:name="android.permission.CAMERA" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.RECORD_AUDIO" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" /&gt;
    &lt;uses-permission android:name="android.permission.BROADCAST_STICKY" /&gt;
    &lt;uses-permission android:name="android.permission.WRITE_SETTINGS" /&gt;
    &lt;!-- 来电消音 --&gt;
    &lt;uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS" &gt;
    &lt;/uses-permission&gt;
    &lt;uses-permission android:name="android.permission.READ_PHONE_STATE" /&gt;
    &lt;uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" /&gt;
&lt;/manifest&gt;

```

 

 

 

 

 

                         