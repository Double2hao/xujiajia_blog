#android 录音笔demo！！！（Baseadapter、file、MediaRecorder、AlertDialog）
录音笔在网上的demo相比记事本就少很多了，当然在功能上与记事本相比也是大相径庭。

记事本涉及到的仅仅是对string 的存储，而且在读取上并不存在什么难点，直接用textview显示便可以了。需要做的主要是使用SQLite对数据进行一个整理。

而录音笔需要考虑的就相对较多了：比**如录音时中断，录音时用户点击播放按钮；未录音，用户点击停止按钮;在录音或者播放时关闭activity；listview的item中需要为button设置不同的点击效果等等。**

**（源码在文章结尾）**

****

为了便于新手学习，在此还是罗列一下涉及的主要知识点：

1、Baseadapter

2、JAVA的file

3、MediaRecorder

4、较多的AlertDialog

5、MediaPlayer



遇到的问题：

**在listview item中的button控件可以获得焦点时,直接为listview设置item长按事件的监听。出现了listview的item长按事件无效的情况。**



解决方法：

直接在Baseadapter中对**该item的布局**进行长按事件的监听（在笔者demo中是linearlayout），也就是说**对item中button的父布局**进行长按事件的监听。





**效果：**

<img src="https://img-blog.csdn.net/20151221085452759?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt="">  <img src="https://img-blog.csdn.net/20151221085502116?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt=""><img src="https://img-blog.csdn.net/20151221085510284?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt="">  <img src="https://img-blog.csdn.net/20151221085517541?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="400" height="700" alt="">





**MainActivity:**



```
package com.example.recorder;

import android.app.Activity;
import android.app.AlertDialog;
import android.app.AlertDialog.Builder;
import android.content.DialogInterface;
import android.media.MediaPlayer;
import android.media.MediaRecorder;
import android.os.Bundle;
import android.os.Environment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.BaseAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

import java.io.File;
import java.io.IOException;

public class MainActivity extends Activity implements OnClickListener {

    private Button start;
    private Button stop;
    private ListView listView;
    ShowRecorderAdpter showRecord;

    // 录音文件播放
    // 录音
    // 音频文件保存地址
    private MediaPlayer myPlayer;
    private MediaRecorder myRecorder = null;
    private String path;
    private File saveFilePath;
    // 所录音的文件名
    String[] listFile = null;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //初始化控件
        InitView();

    }

    private void InitView() {
        start = (Button) findViewById(R.id.start);
        stop = (Button) findViewById(R.id.stop);
        listView = (ListView) findViewById(R.id.list);

        myPlayer = new MediaPlayer();
        showRecord = new ShowRecorderAdpter();

        //如果手机有sd卡
        if (Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED)) {
            try {
                path = Environment.getExternalStorageDirectory()
                        .getCanonicalPath().toString()
                        + "/MyRecorders";
                File files = new File(path);
                if (!files.exists()) {
                    //如果有没有文件夹就创建文件夹
                    files.mkdir();
                }
                listFile = files.list();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        start.setOnClickListener(this);
        stop.setOnClickListener(this);
        listView.setAdapter(showRecord);


    }


    //由于在item中涉及到了控件的点击效果，所以采用BaseAdapter
    class ShowRecorderAdpter extends BaseAdapter {

        @Override
        public int getCount() {
            return listFile.length;
        }

        @Override
        public Object getItem(int arg0) {
            return arg0;
        }

        @Override
        public long getItemId(int arg0) {
            return arg0;

        }

        @Override
        public View getView(final int postion, View arg1, ViewGroup arg2) {
            View views = LayoutInflater.from(MainActivity.this).inflate(
                    R.layout.list_item, null);
            LinearLayout parent = (LinearLayout) views.findViewById(R.id.list_parent);
            TextView filename = (TextView) views.findViewById(R.id.show_file_name);
            Button plays = (Button) views.findViewById(R.id.bt_list_play);
            Button stop = (Button) views.findViewById(R.id.bt_list_stop);

            //在textview中显示的时候把“.amr”去掉
            filename.setText(listFile[postion].substring(0, listFile[postion].length() - 4));
            parent.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View view) {
                    AlertDialog aler = new AlertDialog.Builder(MainActivity.this)
                            .setTitle("确定删除该录音？")
                            .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog
                                        , int which) {
                                    File file = new File(path + "/" + listFile[postion]);
                                    file.delete();
                                    // 在删除文件后刷新文件名列表
                                    File files = new File(path);
                                    listFile = files.list();

                                    // 当文件被删除刷新ListView
                                    showRecord.notifyDataSetChanged();
                                }
                            })
                            .setNegativeButton("取消", null)
                            .create();
                    //设置不允许点击提示框之外的区域
                    aler.setCanceledOnTouchOutside(false);
                    aler.show();
                    return false;
                }
            });
            // 播放录音
            plays.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View arg0) {
                    //确认不是在录音的过程中播放
                    if (myRecorder == null) {
                        try {
                            myPlayer.reset();
                            myPlayer.setDataSource(path + "/" + listFile[postion]);
                            if (!myPlayer.isPlaying()) {
                                myPlayer.prepare();
                                myPlayer.start();
                            } else {
                                myPlayer.pause();
                            }

                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    } else {
                        Toast.makeText(MainActivity.this, "请不要再录音的过程中播放！", Toast.LENGTH_SHORT).show();
                    }
                }
            });
            // 停止播放
            stop.setOnClickListener(new OnClickListener() {

                @Override
                public void onClick(View arg0) {
                    if (myRecorder == null &amp;&amp; myPlayer.isPlaying()) {
                        myPlayer.stop();
                    }
                }
            });
            return views;
        }

    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.start:
                final EditText filename = new EditText(this);
                AlertDialog aler = new Builder(this)
                        .setTitle("请输入要保存的文件名")
                        .setView(filename)
                        .setPositiveButton("确定",
                                new DialogInterface.OnClickListener() {
                                    @Override
                                    public void onClick(DialogInterface dialog,
                                                        int which) {
                                        String text = filename.getText().toString();
                                        //如果文件名为空则跳出提示信息
                                        if (text.equals("")) {
                                            Toast.makeText(MainActivity.this,
                                                    "请不要输入空的文件名!", Toast.LENGTH_SHORT).show();
                                        } else {
                                            //开启录音
                                            RecorderStart(text);

                                            start.setText("正在录音中。。");
                                            start.setEnabled(false);
                                            stop.setEnabled(true);
                                            // 在增添文件后刷新文件名列表
                                            File files = new File(path);
                                            listFile = files.list();
                                            // 当文件增加刷新ListView
                                            showRecord.notifyDataSetChanged();
                                        }
                                    }
                                })
                        .setNegativeButton("取消",null)
                        .create();
                //设置不允许点击提示框之外的区域
                aler.setCanceledOnTouchOutside(false);
                aler.show();
                break;
            case R.id.stop:
                myRecorder.stop();
                myRecorder.release();
                myRecorder = null;
                // 判断是否保存 如果不保存则删除
                aler = new AlertDialog.Builder(this)
                        .setTitle("是否保存该录音")
                        .setPositiveButton("确定", null)
                        .setNegativeButton("取消",
                                new DialogInterface.OnClickListener() {
                                    @Override
                                    public void onClick(DialogInterface dialog,
                                                        int which) {
                                        saveFilePath.delete();
                                        // 在删除文件后刷新文件名列表
                                        File files = new File(path);
                                        listFile = files.list();

                                        // 当文件被删除刷新ListView
                                        showRecord.notifyDataSetChanged();
                                    }
                                }).create();
                //设置不允许点击提示框之外的区域
                aler.setCanceledOnTouchOutside(false);
                aler.show();

                start.setText("录音");
                start.setEnabled(true);
                stop.setEnabled(false);
            default:
                break;
        }

    }

    private void RecorderStart(String text) {
        try {
            myRecorder = new MediaRecorder();
            // 从麦克风源进行录音
            myRecorder.setAudioSource(MediaRecorder.AudioSource.DEFAULT);
            // 设置输出格式
            myRecorder.setOutputFormat(MediaRecorder.OutputFormat.DEFAULT);
            // 设置编码格式
            myRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.DEFAULT);

            String paths = path + "/" + text + ".amr";
            saveFilePath = new File(paths);
            myRecorder.setOutputFile(saveFilePath.getAbsolutePath());
            myRecorder.prepare();
            // 开始录音
            myRecorder.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 如果myPlayer正在播放，那么就停止播放，并且释放资源
        if (myPlayer.isPlaying()) {
            myPlayer.stop();
            myPlayer.release();
        }
        //如果myRecorder有内容（代表正在录音），那么就直接释放资源
        if(myRecorder!=null) {
            myRecorder.release();
            myPlayer.release();
        }
    }

}  
```







```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity"&gt;


    &lt;TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#000"
        android:padding="13dp"
        android:text="语音笔"
        android:textColor="#fff"
        android:textSize="22sp"
        android:textStyle="bold" /&gt;

    &lt;ListView
        android:id="@+id/list"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:padding="10dp"
        &gt;&lt;/ListView&gt;

    &lt;LinearLayout
        android:id="@+id/li1"
        android:padding="10dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"&gt;

        &lt;Button
            android:id="@+id/start"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textSize="20sp"
            android:text="开始录音" /&gt;

        &lt;Button
            android:id="@+id/stop"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:enabled="false"
            android:textSize="20sp"
            android:text="停止录音" /&gt;
    &lt;/LinearLayout&gt;


&lt;/LinearLayout&gt;
```







```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp"
    android:id="@+id/list_parent"
    android:orientation="horizontal"&gt;

    &lt;TextView
        android:id="@+id/show_file_name"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:text="文件名"
        android:textColor="#000"
        android:textSize="20sp"
        /&gt;

    &lt;Button
        android:id="@+id/bt_list_play"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:text="播放" /&gt;

    &lt;Button
        android:id="@+id/bt_list_stop"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:text="停止" /&gt;

&lt;/LinearLayout&gt;  
```



最后附上源码：




