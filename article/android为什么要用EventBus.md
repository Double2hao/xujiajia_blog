#android为什么要用EventBus
# 概览

近期接触到EventBus，发现其对于android开发来说其实是一个很有必要的知识要点，但是之前由于项目限制，一直没有机会使用，也不知其作用。 网上对于EventBus使用详述的文章有很多，本文就不谈使用了，仅谈为什么要用EventBus。

>  
 本文各个例子都是为了方便读者理解所写，但是各场景不一定使用EventBus最为合适。 如有问题，欢迎评论指出。 


# Activity之间通信

一般来讲，Activity之间通信有两种方式：
1. 通过Intent。 比如从一个页面跳转到登录页面，返回是否登录成功。1. 将需要通信的数据先缓存（内存或者硬盘），在后一个Activity的生命周期函数中读取。 比如在“个人信息编辑界面”将数据编辑完毕后，返回“个人信息”界面后刷新数据。1. broadcast、binder、socket、contentProvider等（针对特殊情况使用）
上面几种方式几乎已经可以满足所有的通信需要的，EventBus的出现不是替代他们，而是能让一些复杂的情况变得简单。比如以下这种情况：

1、此时有三个界面：主界面、领金币界面（dialog，背景是主界面）、登录界面。 在领金币界面的时候，背景是主界面。 这种情况中，在登录成功后，主界面是需要有UI更新的，要从未登录状态的UI变成登录状态。 但是，由于当前界面是领金币界面，主界面的onResume和onStart是不会触发的。 在这种情况下，要实现登录后刷新主界面UI的需求就很复杂了。

2、登录界面的开发 前面已经说到登录界面直接可以用intent做。但是在一些情况下也会显得非常复杂。 比如，当前界面嵌套了3层fragment和2层RecyclerView，那么要把Activity中的登录页面数据传到相应的位置也是一个很复杂的过程。

# 数据模块之间通信

一般来说，数据模块之间通信也是两种方式：
1. 通过缓存，将需要通信的数据放到缓存中，在需要的位置取。 比如游戏中获取个人对战记录的时候依赖于个人信息的数据。1. 手动控制 比如切换账号后，“个人信息”、“个人对战记录”、“个人金币信息”都需要刷新，手动控制每个数据模块的数据更新顺序。
针对一些情况，不使用EventBus的话可能实现就会比较复杂。比如以下这种情况：

在切换账号后，除了登录信息以外其他数据的刷新。 倘若将数据的刷新都写在登录界面中，即在登录成功后手动刷新各个数据，那么这就相当于登录界面与其他各个数据模块耦合了。 最优的情况应当是登录界面只负责抛出“切换账号成功”这个事件，然后其他数据模块主动接收这个事件，自己处理自己部分的数据刷新。 而这就可以使用EventBus来实现。

# 多线程之间通信

拿游戏举例。一般游戏都会存在长连接，长连接本身会由于网络问题出现中断或者重连等问题。

一旦发生中断或者重连的情况，页面上需要有所反馈，并且可能不同页面反馈不同，不同游戏反馈不同。因此这定然不能写死在长连接模块中。 理想的情况应当是长连接模块能抛出中断或者重连的事件，然后由需要处理的模块来单独处理。这样不仅简单化了线程间通信的问题，也实现了模块的解耦。

# EventBus注意点
1. 要做好防null，否则很容易会出现crash的情况。 用EventBus的时候，由于事件将由其他模块抛出，因此无法确定什么时候会调用到。1. 在页面结束的时候，记得要unregister。 正常情况下一个页面对Event事件的处理要跟着自己的生命周期走。1. 如果使用的是postSticky事件，那么在事件处理完的时候记得要removeSticky事件，否则事件有可能会触发两次。
```
public class MainActivity extends AppCompatActivity {<!-- -->
    
    private TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {<!-- -->
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        EventBus.getDefault().register(this);
        initViews();
    }

    private void initViews() {<!-- -->
        textView=new TextView(this);
    }
    
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onEvent(LoginEvent loginEvent) {<!-- -->
    	//防null
        if(textView==null){<!-- -->
            return;
        }
        textView.setText("EventBus test");
    }

    @Override
    public void onDestroy() {<!-- -->
        super.onDestroy();
        EventBus.getDefault().unregister(this);
    }
    
  	@Subscribe(threadMode = ThreadMode.MAIN, sticky = true)
 	 public void onEvent(LoginEvent event) {<!-- -->
  	  	//首先处理event事件，处理结束后记得remove
  	  
  	 	 //这里写处理代码
  	  
  	 	 //remove事件
   		 EventBus.getDefault().removeStickyEvent(event);
 	 }
}

```
