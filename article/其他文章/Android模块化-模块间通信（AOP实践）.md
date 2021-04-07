#Android模块化-模块间通信（AOP实践）
# 概述

关于项目模块化之前已经写文章详述过，如对这方面不了解的读者可以参考笔者的这篇文章： 

# 什么是模块间通信

对于一般的项目来说，独立模块之间不会相互依赖，如下例子：

>  
 比如此时有四个模块，主模块，base模块，登录模块，游戏模块。依赖关系应该如下： base模块依赖：无 登录模块依赖：base模块 游戏模块依赖：base模块 主模块依赖：base模块、登录模块、游戏模块 


如上面例子所述。 游戏模块由于没有依赖于登录模块，因此是没法直接调用登录模块的功能的。所以就需要一种方式让游戏模块可以实现自己的需求，调用到需要的登录模块的功能。（当然前提是不能依赖登录模块，否则模块化就没有意义了）

# 模块间如何通信

一般会有两种思路：
1. 依赖注入 给自己的模块提前保留出让其他模块注入功能的接口。 用上面的例子来说，游戏模块可以提前保留出接口。由于主模块同时依赖了登录模块和游戏模块，所以可以在主模块初始化的时候就将游戏模块需要的登录功能注入。
>  
 对依赖注入有兴趣的读者可以看笔者的这篇文章： 

1. 面向切片编程（AOP） 每个模块提前保留出提供给其他模块调用的接口。 用上面的例子来说，base模块提前写好供其他模块注册与调用的接口。由于登录模块和游戏模块都依赖了base模块，因此都可以调用到相关功能。
# AOP 实践

>  
 此例子更多是给读者一个思路，如要实际使用还需拓展优化。 


本文主要以AOP的思路为例做了实现，主要内容如下：
1. 包括，app模块，core模块，login模块1. core模块：实现供其他模块注册与调用的接口1. login模块：实现调起登录页面的接口，并且会在登录成功后调用被注册的callback。1. app模块：调用调起登录页面的接口，注册登录成功后的回调。
## app模块

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/20.png" alt="在这里插入图片描述">

### MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    ModuleCenter.getInstance().callFunction("startLogin", null, this);

    ModuleCenter.getInstance().registerCallback("login_callback", new ModuleCallback() {<!-- -->
      @Override public ModuleResult callback(String json, Object object) {<!-- -->
        //callback内容
        //如果有返回内容则通过ModuleResult传递

        return null;
      }
    });
  }
}

```

## login模块

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/21.png" alt="在这里插入图片描述">

### LoginActivity.java

```
public class LoginActivity extends Activity {<!-- -->

  @Override protected void onCreate(@Nullable Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
  }

  public void LoginSuccess() {<!-- -->
    ModuleCenter.getInstance().callCallback("login_callback", null, null);
  }
}

```

### ModuleProvider.java

```
public class ModuleProvider {<!-- -->

  private static void initProvider(){<!-- -->
    ModuleCenter.getInstance().registerFunction("startLogin", new ModuleFunction() {<!-- -->
      @Override public void function(String json,Object object) {<!-- -->
        Context context=(Context)object;
        Intent intent=new Intent(context, LoginActivity.class);
        context.startActivity(intent);
      }
    });

  }
}

```

## core模块

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/22.png" alt="在这里插入图片描述">

### ModuleCenter.java

```
public class ModuleCenter {<!-- -->
  private HashMap&lt;String, ModuleFunction&gt; mapModuleFunction;
  private HashMap&lt;String, List&lt;ModuleCallback&gt;&gt; mapModuleCallback;

  private static class Host {<!-- -->
    private static ModuleCenter instance = new ModuleCenter();
  }

  private ModuleCenter() {<!-- -->
    mapModuleFunction = new HashMap&lt;&gt;();
    mapModuleCallback = new HashMap&lt;&gt;();
  }

  public static ModuleCenter getInstance() {<!-- -->
    return Host.instance;
  }

  public void registerFunction(String type, ModuleFunction moduleFunction) {<!-- -->
    mapModuleFunction.put(type, moduleFunction);
  }

  public void unRegisterFunction(String type) {<!-- -->
    mapModuleFunction.remove(type);
  }

  public void registerCallback(String type, ModuleCallback moduleCallback) {<!-- -->
    List&lt;ModuleCallback&gt; listCallback = mapModuleCallback.get(type);
    if (listCallback == null) {<!-- -->
      listCallback = new ArrayList&lt;&gt;();
    }
    listCallback.add(moduleCallback);
  }

  public void unRegisterCallback(String type, ModuleCallback moduleCallback) {<!-- -->
    List&lt;ModuleCallback&gt; listCallback = mapModuleCallback.get(type);
    if (listCallback != null) {<!-- -->
      listCallback.remove(moduleCallback);
    }
  }

  public void callFunction(String type, String json, Object object) {<!-- -->
    ModuleFunction moduleFunction = mapModuleFunction.get(type);
    if (moduleFunction != null) {<!-- -->
      moduleFunction.function(json, object);
    }
  }

  public void callCallback(String type, String json, Object object) {<!-- -->
    List&lt;ModuleCallback&gt; list = mapModuleCallback.get(type);
    if (list != null) {<!-- -->
      for (ModuleCallback moduleCallback : list) {<!-- -->
        moduleCallback.callback(json, object);
      }
    }
  }
}

```

### ModuleResult.java

```
public class ModuleResult {<!-- -->
  private String jsonResult;

  public String getJsonResult() {<!-- -->
    return jsonResult;
  }

  public void setJsonResult(String jsonResult) {<!-- -->
    this.jsonResult = jsonResult;
  }
}

```

### ModuleFunction.java

```
public interface ModuleFunction {<!-- -->
  void function(String json, Object object);
}

```

### ModuleCallback.java

```
public interface ModuleCallback {<!-- -->
  ModuleResult callback(String json, Object object);
}

```