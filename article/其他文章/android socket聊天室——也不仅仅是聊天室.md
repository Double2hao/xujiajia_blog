#android socket聊天室——也不仅仅是聊天室
# 前提概要

笔者很久之前其实就已经学习过了socket，当然也是用socket做过了聊天室，但是觉得此知识点比较一般，并无特别难的技术点，于是也并未深究。 然而近期一个项目中对socket的使用却让笔者感觉socket强大无比，可以实现诸多功能。

# 个人Socket体验

项目主要有关智能家居，需要实现多台手机同时对灯进行操作（开或者关），主要需要实现以下几点： 1、进入界面时获取所有灯的状态。 2、一台手机改变了灯的状态，其他的手机上可以有所显示。 3、硬件上改变了灯的状态（手动开关灯），所有手机上要有所显示。

此功能如果使用HTTP读取的方式实现就不太合适了。一方面客户端与服务器读取文件的同步性难以保证，即使保证了，也需要浪费大量性能；另一方面，类似笔者的这种项目功能服务器和客户端交互比较频繁，对“即时性”要求也比较高，用HTTP不仅性能消耗太大，而且难以保证“即时性”。

但是使用Socket就很容易实现了，主要逻辑如下： 1、每次进入界面与服务器建立Socket连接，并得到此时灯的状态 2、每次需要对灯进行操作的时候建立一个线程把灯的状态传递给服务器，服务器接收到之后，把该状态传递给每一个此时与服务器建立连接的客户端。

此次体验也是让笔者想起了学长之前做的一道笔试题，题目大概如下：

>  
 将淘宝网页和手机版同时打开账户，手机停留在购物车界面，此时网页上将某一物品加入购物车，如何设计才能让手机自动刷新购物车。 


如果使用socket，相信是一个不错的思路。

好了，接下来进入正题，展示socket聊天室demo。

# 效果（源码在文章结尾）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040178890.png" alt="这里写图片描述">

# 主要思路

## Android

1、进入界面客户端与服务器建立socket，同时此时开启一个线程一直接收服务器发送来的消息。 2、每次点击button获取EditText中的字符串，调用子线程把字符串发送给服务器。

## 服务器

1、创建一个ArrayList存储Socket。 2、循环接收请求访问该端口的客户端，接收到之后，把该socket存储到ArrayList中，并且为每一个socket开启一个线程用于通信。 3、每个socket的线程的逻辑如下：循环接收客户端发来的消息，接收到之后，利用之前的ArrayList，发送到每一个客户端。如果某个客户端返回空值或者无法发送过去，那么表示该客户端已经断开，就从ArrayList中移除。

# 代码

>  
 （借鉴《Android疯狂讲义》） 


## Android

>  
 不要忘记在AndroidManifest里面加上访问网络的权限 


### MainActivity:

```
package com.example.double2.sockettesttwo;

import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    private EditText etMain;
    private Button btnMain;
    private TextView tvMain;
    private ClientThread mClientThread;

    //在主线程中定义Handler传入子线程用于更新TextView
    private Handler mHandler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etMain = (EditText) findViewById(R.id.et_main);
        btnMain = (Button) findViewById(R.id.btn_main);
        tvMain = (TextView) findViewById(R.id.tv_main);

        mHandler=new Handler() {
            @Override
            public void handleMessage(Message msg) {
                if (msg.what == 0) {
                    tvMain.append("\n" + msg.obj.toString());
                }
            }
        };

        //点击button时，获取EditText中string并且调用子线程的Handler发送到服务器
        btnMain.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                try {
                    Message msg = new Message();
                    msg.what = 1;
                    msg.obj = etMain.getText().toString();
                    mClientThread.revHandler.sendMessage(msg);
                    etMain.setText("");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });


        mClientThread = new ClientThread(mHandler);
        new Thread(mClientThread).start();


    }
}


```

### ClientThread

```
package com.example.double2.sockettesttwo;

import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.util.Log;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.Socket;

/**
 * 项目名称：SocketTestTwo
 * 创建人：Double2号
 * 创建时间：2016.11.20 9:16
 * 修改备注：
 */
public class ClientThread implements Runnable {
    private Socket mSocket;
    private BufferedReader mBufferedReader = null;
    private OutputStream mOutputStream = null;
    private Handler mHandler;

    public Handler revHandler;

    public ClientThread(Handler handler) {
        mHandler = handler;
    }

    @Override
    public void run() {
        try {
            mSocket = new Socket("10.3.20.159", 30003);
            Log.d("xjj","connect success");
            mBufferedReader = new BufferedReader(new InputStreamReader(mSocket.getInputStream()));
            mOutputStream = mSocket.getOutputStream();

            new Thread(){
                @Override
                public void run() {
                    super.run();
                    try {
                        String content = null;
                        while ((content = mBufferedReader.readLine()) != null) {
                            Log.d("xjj",content);
                            Message msg = new Message();
                            msg.what = 0;
                            msg.obj = content;
                            mHandler.sendMessage(msg);
                        }
                    }catch (IOException e){
                        e.printStackTrace();
                    }
                }
            }.start();

            //由于子线程中没有默认初始化Looper，要在子线程中创建Handler，需要自己写
            Looper.prepare();
            revHandler = new Handler() {
                @Override
                public void handleMessage(Message msg) {
                    if (msg.what == 1) {
                        try {
                            mOutputStream.write((msg.obj.toString() + "\r\n").getBytes("utf-8"));
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }
            };
            Looper.loop();





        } catch (IOException e) {
            e.printStackTrace();
            Log.d("xjj","");
        }
    }
}


```

### activity_main

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="@dimen/activity_vertical_margin"
    android:orientation="vertical"
    &gt;

    &lt;LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"&gt;

        &lt;EditText
            android:id="@+id/et_main"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"/&gt;

        &lt;Button
            android:id="@+id/btn_main"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/send"/&gt;
    &lt;/LinearLayout&gt;

    &lt;TextView
        android:id="@+id/tv_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        /&gt;
&lt;/LinearLayout&gt;


```

## 服务器：

### MySever

```
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.ArrayList;

public class MySever {
	
	public static ArrayList&lt;Socket&gt; socketList = new ArrayList&lt;Socket&gt;();
	public static String content="";

	public static void main(String[] args) throws IOException {
		//建立ServerSocket
		ServerSocket ss = new ServerSocket(30003);
		
		//不断接收此端口的socket，并存储到socketList中
		//并且为每一个socket开启一个线程，用于接收信息
		while (true) {
			Socket s = ss.accept();
			System.out.println("connect success!");
			socketList.add(s);

			new Thread(new ServerThread(s)).start();
		}
	}

}


```

### SeverThread

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.UnsupportedEncodingException;
import java.net.Socket;
import java.net.SocketException;
import java.util.Iterator;

public class ServerThread implements Runnable {

	private Socket mSocket = null;
	private BufferedReader mBufferedReader = null;

	// 构造函数中接收socket并且初始化BufferedReader
	public ServerThread(Socket socket) 
			throws UnsupportedEncodingException, IOException {
		mSocket = socket;
		mBufferedReader = new BufferedReader(
				new InputStreamReader(mSocket.getInputStream(), "utf-8"));
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub

		try {
			String content = null;
			
			//循环接收来自此客户端的消息
			//如果接收不到了，表面已经断开，就将此客户端从socketList中移除
			while ((content = mBufferedReader.readLine()) != null) {

				System.out.println(content);
				
				//向连接中的每个客户端发送数据
				//如果异常，说明断开，就将该条目从socketList中移除
				for (Iterator&lt;Socket&gt; it = MySever.socketList.iterator(); 
						it.hasNext();) {
					Socket s = it.next();
					try {
						OutputStream os = s.getOutputStream();
						os.write((content + "\n").getBytes("utf-8"));
					} catch (SocketException e) {
						e.printStackTrace();
						it.remove();
					}
				}
			}
		} catch (IOException e) {
			e.printStackTrace();
			MySever.socketList.remove(mSocket);
		}
	}

}


```

## 源码地址：

