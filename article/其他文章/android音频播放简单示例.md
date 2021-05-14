#android音频播放简单示例
此知识点比较简单，直接上图和代码：

 

<img alt="" class="has" height="700" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911403440.png " width="400">

 

 

MainActivity:

 

```
import java.io.File;

import android.app.Activity;
import android.media.MediaPlayer;
import android.os.Bundle;
import android.os.Environment;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity implements OnClickListener {

	private Button play;

	private Button pause;

	private Button stop;

	private MediaPlayer mediaPlayer = new MediaPlayer();

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		play = (Button) findViewById(R.id.play);
		pause = (Button) findViewById(R.id.pause);
		stop = (Button) findViewById(R.id.stop);
		play.setOnClickListener(this);
		pause.setOnClickListener(this);
		stop.setOnClickListener(this);
		initMediaPlayer();//初始化MediaPlayer
	}

	//初始化MediaPlayer
	private void initMediaPlayer() {
		try {
			//首先通过File对象指定音频文件的路径
			//此处示例中sd卡中的文件名为“music.mp3”
			File file = new File(Environment.getExternalStorageDirectory(), "music.mp3");
			mediaPlayer.setDataSource(file.getPath());
			//让medieplayer进入准备状态
			mediaPlayer.prepare();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.play:
			if (!mediaPlayer.isPlaying()) {
				mediaPlayer.start();//开始播放
			}
			break;
		case R.id.pause:
			if (mediaPlayer.isPlaying()) {
				mediaPlayer.pause();//暂停播放
			}
			break;
		case R.id.stop:
			if (mediaPlayer.isPlaying()) {
				//此处是将mediaplayer重置到刚刚创建的状态，也就是还没有设置文件路径的状态
				mediaPlayer.reset();//停止播放
				//重新调用重置方法
				initMediaPlayer();

				//此处也是可以直接调用stop（）方法，用reset仅仅为了演示
			}
			break;
		default:
			break;
		}
	}

	//在activity被摧毁的时候将mediaPlayer停止并且释放掉
	@Override
	protected void onDestroy() {
		super.onDestroy();
		if (mediaPlayer != null) {
			mediaPlayer.stop();
			mediaPlayer.release();
		}
	}

}
```

 

 

 

 

 

activity_main:

 

```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" &gt;

    &lt;Button
        android:id="@+id/play"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Play" /&gt;

    &lt;Button
        android:id="@+id/pause"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Pause" /&gt;

    &lt;Button
        android:id="@+id/stop"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Stop" /&gt;

&lt;/LinearLayout&gt;
```

 

 