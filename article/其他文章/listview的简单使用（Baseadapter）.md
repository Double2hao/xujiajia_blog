#listview的简单使用（Baseadapter）
本篇主要是写下listview的简单实现。（使用Baseadapter）

在自己理解之后稍微做下笔记，在Baseadapter的部分增添了较多注释。

也是希望能给和我一样的新手们一些帮助。

 

**以下是实现效果：**

**<img alt="" class="has" height="476" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2980.png" width="268">**

 

 

 

**<u>接下来是代码：</u>**

 

**<u>main.xml</u>**

 

>  
 &lt;?xml version="1.0" encoding="utf-8"?&gt; &lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="fill_parent" android:layout_height="fill_parent" android:orientation="vertical" &gt; 
   
 &lt;ListView android:id="@+id/lv" android:layout_width="fill_parent" android:layout_height="wrap_content" android:fastScrollEnabled="true" /&gt; 
   
 &lt;/LinearLayout&gt; 


 

**<u>list_item.xml</u>**

 

>  
 &lt;?xml version="1.0" encoding="utf-8"?&gt; &lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:orientation="horizontal" &gt; 
   
 &lt;ImageView android:id="@+id/img" android:layout_width="wrap_content" android:layout_height="wrap_content" /&gt; &lt;LinearLayout  android:layout_width="fill_parent" android:layout_height="wrap_content" android:orientation="vertical" &gt; &lt;TextView android:id="@+id/tv" android:layout_width="wrap_content" android:layout_height="wrap_content" android:textSize="20sp" /&gt; &lt;TextView  android:id="@+id/info" android:layout_width="wrap_content" android:layout_height="wrap_content" android:textSize="14sp" /&gt; &lt;/LinearLayout&gt; 
   
 &lt;/LinearLayout&gt; 
   


 

**<u>Demo17Activity.java</u>**

>  
 package com.example.baseadapter1; 
 import java.util.ArrayList; import java.util.HashMap; import java.util.List; import java.util.Map; 
 import android.app.Activity; import android.content.Context; import android.os.Bundle; import android.view.LayoutInflater; import android.view.View; import android.view.ViewGroup; import android.widget.BaseAdapter; import android.widget.ImageView; import android.widget.ListView; import android.widget.TextView; 
 public class Demo17Activity extends Activity {<!-- --> private ListView lv; private List&lt;Map&lt;String,Object&gt;&gt; data; @Override public void onCreate(Bundle savedInstanceState) {<!-- --> super.onCreate(savedInstanceState); setContentView(R.layout.main); lv=(ListView)findViewById(R.id.lv); data=getdata(); MyAdapter adapter=new MyAdapter(this); lv.setAdapter(adapter);     
 **//这里一定要注意，setAdapter的作用是将listview与adapter绑定，而并非是使用这个listview展示出此时adapter中的内容。也就是说，在<strong>setAdapter之后，倘若修改<strong>adapter中的内容，listview**</strong>所展示的内容就会改变。</strong> 
  } private List&lt;Map&lt;String,Object&gt;&gt; getdata() {<!-- --> List&lt;Map&lt;String,Object&gt;&gt; list=new ArrayList&lt;Map&lt;String,Object&gt;&gt;(); for(int i=0;i&lt;10;i++) {<!-- --> Map&lt;String,Object&gt; map=new HashMap&lt;String, Object&gt;(); map.put("img", R.drawable.ic_launcher); map.put("title", "hehehehehe"); map.put("info", "66666666"); list.add(map); } return list; } static class ViewHolder {<!-- --> public ImageView img; public TextView title; public TextView info; } public class MyAdapter extends **BaseAdapter**{<!-- --> private LayoutInflater mInflater=null; public **MyAdapter**(Context context) {<!-- --> // TODO Auto-generated constructor stub this.mInflater=LayoutInflater.from(context);  **//这里就是确定你listview在哪一个layout里面展示** } @Override public int **getCount()** {<!-- --> // TODO Auto-generated method stub return data.size(); **//这个决定你listview有多少个item** } @Override public Object **getItem**(int position) {<!-- --> // TODO Auto-generated method stub return position; } @Override public long **getItemId**(int position) {<!-- --> // TODO Auto-generated method stub return position; } @Override public View **getView**(int position, View convertView, ViewGroup parent) {<!-- --> // TODO Auto-generated method stub ViewHolder holder=null;  if(convertView==null) {<!-- --> holder=new ViewHolder(); convertView=mInflater.inflate(R.layout.list_item,null); **//这里确定你listview里面每一个item的layout** holder.img = (ImageView)convertView.findViewById(R.id.img);** //此处是将内容与控件绑定。** 
 holder.title = (TextView)convertView.findViewById(R.id.tv);**//注意：此处的findVIewById前要加convertView.** holder.info = (TextView)convertView.findViewById(R.id.info); convertView.setTag(holder); } else{<!-- --> holder=(ViewHolder)convertView.getTag(); **//这里是为了提高listview的运行效率** } holder.img.setBackgroundResource((Integer)data.get(position).get("img")); holder.title.setText((String)data.get(position).get("title")); holder.info.setText((String)data.get(position).get("info")); return convertView; } } } 
