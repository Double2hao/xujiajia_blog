#android 注解入门（Acitivity路由demo）
>  
 参考《Java编程思想》 


# 概述

近期接触了路由和模块间通信的内容，发现Java注解非常常用。 避免后面看各源码被其阻塞，大致了解了下，作此文记录之。

>  
 本文的Demo是 运行时注解的Demo。 编译时注解将在其他文章讲述，本文不作具体阐述。 


# 注解类型
- @Target- @Retention- @Documented- @Inherited
##### @Target

用于描述注解的使用范围，可能的ElementType参数如下：
- CONSTRUCTOR:用于描述构造器- FIELD:用于描述域- LOCAL_VARIABLE:用于描述局部变量- METHOD:用于描述方法- PACKAGE:用于描述包- PARAMETER:用于描述参数- TYPE:用于描述类、接口(包括注解类型) 或enum声明
##### @Retention

表示需要在什么级别保存该注释信息。可选的RetentionPolicy参数如下：
- SOURCE:在源文件中有效，当Java文件编译成class文件的时候，注解被遗弃。（比如@AutoService）- CLASS:在class文件中有效，jvm加载class文件时候被遗弃。（可以用来动态生成java文件等）- RUNTIME:在运行时有效，jvm加载class文件之后，仍然存在。（运行时可以通过反射获取）
##### @Documented

表示此注解 记录在了java文档中。自定义不会使用。

##### @Inherited

允许子类继承父类中的注解

# Demo——Activity路由

Activity路由是一个比较常见会使用到注解的场景。 此处为了演示，代码非常简略，实际实现路由不会这么简单。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910371600.png " width="50%" height="50%">

##### 最终输出log：

```
com.example.annotaiontest D/RouterManager: {com.example.annotaiontest.MainActivity=scheme://test}

```

### MainActivity.java

使用注解，并且在RouterManager初始化后输出注册的内容。

```
@TestRouter(url = "scheme://test")
public class MainActivity extends AppCompatActivity {<!-- -->

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    RouterManager.getInstance().init();//初始化
    RouterManager.getInstance().showAllActivity();//show目前注册的
  }
}

```

### TestRouter.java

自定义注解

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestRouter {<!-- -->
  public String url() default "";
}

```

### RouterManager.java

用于存储注册的路由。初始化的时候会执行注册操作。

```
public class RouterManager {
  //data
  private ArrayMap&lt;String, String&gt; map = new ArrayMap&lt;&gt;();

  private static final class Host {
    private static final RouterManager instance = new RouterManager();
  }

  private RouterManager() {
  }

  public void init() {
    RouterManager.getInstance().register(MainActivity.class);
  }

  public static RouterManager getInstance() {
    return Host.instance;
  }

  public void register(Class&lt;? extends Activity&gt; clazz) {
    TestRouter testEvent = clazz.getAnnotation(TestRouter.class);//尝试获取注解对象
    if (testEvent != null) {
      map.put(clazz.getName(), testEvent.url());
    }
  }

  public void showAllActivity() {
    Log.d("RouterManager", map.toString());
  }
}

```