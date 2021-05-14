#android  socket简单编程（java在PC本地创建服务器）
笔者此文也不太详述socket的原理了，主要是给新手一个demo，能够更好地理解android上的socket。



主要内容：

1、在java上搭建一个简单服务器接收数据

2、在android上建立客户端传递数据



最终接收到数据效果：



<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911646430.png " alt="">





本文已经说了是**使用java在PC本地创建服务器，**首先肯定是要获取本地的IP地址，百度上方法有很多，笔者使用的是在cmd框中输入**ipconfig | findstr IPv4**或者**ipconfig/all**查看，具体效果如图（图上是笔者的IP）：



<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911647801.png " alt="">





然后实现服务器端程序，笔者是在eclipse中创建一个项目，直接定义一个JAVA类，如下：





```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.net.Socket;


public class SimpleServer {
	public static void main(String[] args)
	throws IOException
	{
		System.out.println("start while22");
		//创建一个ServerSocket，用于监听客户端Socket的连接请求
		ServerSocket ss=new ServerSocket(30000);
		while(true)
		{
			//每当接收到客户端Socket请求的时候，服务器端也对应产生一个Socket
			//将Socket对应的输入流包装秤ButtferedReader
            //进行普通IO操作
			//关闭流，关闭Socket
			Socket socket=ss.accept();
			
			BufferedReader in = new BufferedReader(
					new InputStreamReader(socket.getInputStream()));  
            String str = in.readLine();  
            System.out.println("来自客户端的数据为:" + str);   
            
            in.close();
            socket.close();
		}
	}
}

```



接着再实现客户端代码：

**此处一定要用虚拟机，一定要用虚拟机，****一定要用虚拟机。如果用真机运行客户端是找不到你的本地服务器的，**笔者在此点上折腾了好多时间，期间一直在怀疑是自己的代码写错了。。。。

**MainActivity：**



```
import android.app.Activity;
import android.content.DialogInterface;
import android.os.Bundle;
import android.os.Message;
import android.support.design.widget.FloatingActionButton;
import android.support.design.widget.Snackbar;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.View;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.Socket;
import java.util.logging.Handler;
import java.util.logging.LogRecord;

public class MainActivity extends Activity implements View.OnClickListener {

    private Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button=(Button)findViewById(R.id.button);
        button.setOnClickListener(this);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.button:
                new Thread()
                {
                    @Override
                    public void run() {
                        super.run();
                        try {
                            //创建Socket
                            Socket socket=new Socket("10.3.18.49",30000);
                            //向服务器发送消息
                            //建立输出流
                            OutputStream out=socket.getOutputStream();
                            out.write("1111111111".getBytes("utf-8"));
                            //关闭流
                            out.close();
                            //关闭Socket
                            socket.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }.start();
                break;
        }
    }

}

```





```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:orientation="vertical"
    &gt;

    &lt;Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="SocketTest" /&gt;
&lt;/LinearLayout&gt;
```





```
 &lt;uses-permission android:name="android.permission.INTERNET"/&gt;
```





末尾是贴上笔者实现的时候的一些图，希望有助于读者理解：



实现的时候打开了，android studio，虚拟机，以及eclipse。（笔者电脑一般，特别卡。。）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911648502.png " alt="">



由于是为了测试，所以activity布局比较简单，就只有一个button来发送数据。



<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911650223.png " alt="">



