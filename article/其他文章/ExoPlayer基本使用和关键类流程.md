#ExoPlayer基本使用和关键类流程
>  
 官网地址：http://google.github.io/ExoPlayer/ 


# 例子

极简的逻辑，界面上仅仅显示一个SimpleExoPlayerView。

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910601320.png " alt="这里写图片描述">

## 添加依赖

```
repositories {
    jcenter()
    google()
}

```

```
compile 'com.google.android.exoplayer:exoplayer:2.6.0'


```

## 代码

>  
 不要忘记在AndroidManifest中添加网络权限。 


MainActivity：

```
public class MainActivity extends AppCompatActivity {

    private final String testUrl="http://119.84.101.207/videos/v0/20180101/9c/20/33369eec370be393dd555a5a20234c02.mp4?key=0ebb94883d2df6eeb873b2dd48a35f687&amp;dis_k=2d8cd483e5d3cf71159fcdfddad137350&amp;dis_t=1514877572&amp;dis_dz=CT-QIYI_SHMinRun&amp;dis_st=44&amp;src=iqiyi.com&amp;uuid=a795aea-5a4b3284-bd&amp;m=v&amp;qd_ip=65e30cfd&amp;qd_p=65e30cfd&amp;qd_k=ab6b3e8679e84cccd49bfc91d5975606&amp;qd_src=02028001010000000000&amp;ssl=1&amp;ip=101.227.12.253&amp;qd_vip=0&amp;dis_src=vrs&amp;qd_uid=0&amp;qdv=1&amp;qd_tm=1514877572862";
    SimpleExoPlayer mPlayer;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        initExoPlayer();
    }

    private void initExoPlayer() {
        RenderersFactory renderersFactory=new DefaultRenderersFactory(this);
        DefaultTrackSelector trackSelector=new DefaultTrackSelector();
        LoadControl loadControl=new DefaultLoadControl();
        mPlayer= ExoPlayerFactory.newSimpleInstance(renderersFactory, trackSelector,loadControl);

        SimpleExoPlayerView playerView=new SimpleExoPlayerView(this);
        playerView.setPlayer(mPlayer);
        setContentView(playerView);

        Uri mp4Uri=Uri.parse(testUrl);
        DefaultDataSourceFactory dataSourceFactory=new DefaultDataSourceFactory(
                this, Util.getUserAgent(this,"exoPlayerTest"));
        ExtractorsFactory extractorsFactory=new DefaultExtractorsFactory();
        MediaSource mediaSource=new ExtractorMediaSource(
                mp4Uri,dataSourceFactory,extractorsFactory,null,null);
        mPlayer.prepare(mediaSource);
    }


    @Override
    protected void onDestroy() {
        mPlayer.release();
        super.onDestroy();
    }
}


```

# 关键类流程图

>  
 如果对播放器原理完全不理解的同学可以看下此文章：https://www.jianshu.com/p/82e778eb618b 


<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910604051.png " alt="这里写图片描述">