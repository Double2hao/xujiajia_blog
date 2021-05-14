#MVC、MVP、MVVM小记
>  
 本文参考  


# 前提概要

MVC、MVP、MVVM是三个最常见的软件架构模式中，平时与其他人的交流中也经常涉及到。 细想的时候发现自己对这些其实也只是略知一二，并未真正的了解总结过，于是作此文。 此文对个模式概念的描述不多，主要是用例子来说明。

# MVC（Model-View-Controller）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911593890.png " alt="在这里插入图片描述"> **Model：** 数据结构和数据结构相关的类，比如说网络数据获取逻辑，本地数据存储逻辑等。 **View：** 视图层，在前端就是html，android上就是xml。 **Controller：** 控制层，android上就是Activity，他主要负责业务逻辑，通过Controller可以将Model中的数据放到View上，也是通过Controller使我们对View的操作能在Model中有所体现。（比如点击后刷新网络数据之类）

# MVP（Model+View+Presenter）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911601311.png " alt="在这里插入图片描述"> MVP 架构模式是基于MVC来的，当MVC中Controller处理所有的业务逻辑的时候，随着项目的增大，会发现View和Model的代码在Controller中耦合了起来。Presenter就担任了给View和Model解耦的角色。（具体可以看后面的例子）

>  
 个人认为MVP模式应该称为MVPC模式，因为Presenter并不是替代了Controller的作用，它的主要作用是给View和Model解耦，Controller直接与Presenter交互，这样View和Model就不会随着业务的增大而越来越耦合。 


# MVVM（Model+View+ViewModel）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911606592.png " alt="在这里插入图片描述"> MVVM和MVC就没啥关系了，要理解MVVM先可以去了解一下前端流行框架的VDOM。 也可以参考笔者之前的文章：

将前端的各个模块用MVVM来对应： Model：JS代码 View：DOM （一般是html+CSS） ViewModel：VDOM

# 用例子讲一下MVC和MVP（Android例子）

>  
 例子：现在页面显示用户信息，用户信息从数据库中获取。 


### MVC

首先我们使用MVC的方式来写项目。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911609703.png " alt="在这里插入图片描述"> Model：UserInfo.java + DataBaseWork.java Controller：MainActivity.java View：activity_main.xml

**代码如下：** UserInfo.java

```
public class UserInfo {
    private String name;
    private String age;
    
    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getAge() {
        return age;
    }
}

```

DataBaseWork.java

```
public class DataBaseWork {

    public static UserInfo getUserInfoFromDataBase(){
        UserInfo userInfo=new UserInfo();
        //此处执行数据库操作

        return userInfo;
    }
}

```

activity_main.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"&gt;

    &lt;TextView
        android:id="@+id/tv_main"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" /&gt;

&lt;/RelativeLayout&gt;

```

MainActivity.java

```
public class MainActivity extends Activity {

    private TextView tvMain;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvMain=findViewById(R.id.tv_main);
        tvMain.setText(DataBaseWork.getUserInfoFromDataBase().getName());
    }
}

```

这样就算是一个简单的MVC项目了。 随着业务的发展，我们发现每次都从数据库中去取数据不是很有效率，于是打算做一个本地缓存，于是就添加了一个专门控制缓存的模块，即CacheManager.java。 CacheManager.java

```
public class CacheManager {
    public static UserInfo getUserInfoFromCache(){
        UserInfo userInfo=new UserInfo();
        //此处执行数据库操作

        return userInfo;
    }
}

```

MainActivity.java的代码就更新成了如下，首先去获取缓存，如果没有缓存的话就从数据库中读取。

```
public class MainActivity extends Activity {

    private TextView tvMain;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvMain=findViewById(R.id.tv_main);

        UserInfo userInfo;
        userInfo=CacheManager.getUserInfoFromCache();
        if(userInfo==null){
            userInfo=DataBaseWork.getUserInfoFromDataBase();
        }
        tvMain.setText(userInfo.getName());
    }
}

```

业务更新到这里，细心思考一下，我们就可以发现View和Model的逻辑已经耦合起来了，导致如果我们Model有什么新的功能的添加，我们不得不要到Controller中去更改View的更新逻辑。 试想一下，如果上面例子中使用“DataBaseWork.getUserInfoFromDataBase()”的模块不仅仅只有ManActivity着一处，而有十几处甚至更多的地方使用到了，那么添加缓存的逻辑就会变得极其复杂，而且引入了较大的风险。

至此，MVP就可以出场解决这样的问题了：

### MVP

MVP最终代码结构: <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911614734.png " alt="在这里插入图片描述"> MVP中的Presenter就是此处的UserInfoDataManager，代码如下：

```
public class UserInfoDataManager {
    public static UserInfo getUserInfo(){
        UserInfo userInfo;
        userInfo=CacheManager.getUserInfoFromCache();
        if(userInfo==null){
            userInfo=DataBaseWork.getUserInfoFromDataBase();
        }
        return userInfo;
    }
}

```

MainActivity.java的代码更新如下：

```
public class MainActivity extends Activity {

    private TextView tvMain;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvMain=findViewById(R.id.tv_main);
        tvMain.setText(UserInfoDataManager.getUserInfo().getName());
    }
}

```

这样，MainActivity作为Controller要获取数据的时候只需要和Presenter交互了，并不需要去管理Model的获取数据的逻辑，因此Model也不存在会和View耦合的问题了。

还是拿这个例子来说，倘若之后又有一个需求：

>  
 当数据库中获取不到数据的时候，要到网络上把数据下载到数据库中。 


MainActivity的代码也不需要更改了，我们只需要在Model中添加网络获取数据的逻辑，然后再更新UserInfoDataManager.getUserInfo()这个方法就可以了。 如果说项目较大，在整个项目中调用“UserInfoDataManager.getUserInfo()”这个方法的地方有十几处甚至更多，我们也不用去花时间了解其他模块，不仅节省了时间成本，也避免了承担额外的风险。