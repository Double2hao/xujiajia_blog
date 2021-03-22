#编译时注解 AbstractProcessor （Activity路由Demo）
# 概述

前一篇文章已经整理过注解的一些概念，也是附上了运行时注解的Demo，如果对注解概念不是很熟的读者建议先看下前一篇文章：

此篇文章主要讲一下编译时注解的使用，同时也是以”Activity路由“的Demo为例子。

>  
 本篇的Demo主要是演示了使用编译时注解来创建文件的功能。 


# 主要模块
1. **anotationrouter**：创建注解
1. **processortest**：自定义注解解释器，即实现AbstractProcessor。
1. **app**：使用注解

# 创建注解

##### TestRouter

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)//设置为编译时生效
public @interface TestRouter {<!-- -->
  public String url() default "";
}

```

##### RouteManager

```
public class RouterManager {<!-- -->

  public static final String INIT_CLASS = "com.example.generated.TestRouterInit";
  public static final String INIT_METHOD = "init";

  private HashMap&lt;String, String&gt; map = new HashMap&lt;&gt;();

  private static final class Host {<!-- -->
    private static final RouterManager instance = new RouterManager();
  }

  private RouterManager() {<!-- -->
  }

  public void init() {<!-- -->
    try {<!-- -->
      Class.forName(INIT_CLASS).getMethod(INIT_METHOD).invoke(null);//调用动态生成的文件
    } catch (Exception e) {<!-- -->
      e.printStackTrace();
    }
  }

  public static RouterManager getInstance() {<!-- -->
    return Host.instance;
  }

  public void register(String key, String uri) {<!-- -->
    if (key != null &amp;&amp; uri != null) {<!-- -->
      map.put(key, uri);
    }
  }

  public void showAllActivity() {<!-- -->
    System.out.println("RouterManager:" + map.toString());
  }
}

```

# 实现AbstractProcessor

##### build.gradle(processortest模块)

主要引入了几个依赖：
- annotationrouter：用于引入demo中定义的注解
- javapoet：用于动态生成文件
- auto-service：用来生成META-INF/services/javax.annotation.processing.Processor文件
 <img src="https://img-blog.csdnimg.cn/20200322161739577.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" width="50%" height="50%">
>  
 这里为什么auto-service同时需要compileOnly和annotationProcessor？ compileOnly：为了在源文件中可以使用@AutoService annotationProcessor：为了触发注解解释器 


```
apply plugin: 'java-library'

dependencies {
  implementation project(':annotationrouter')
  implementation 'com.squareup:javapoet:1.12.1'
  compileOnly 'com.google.auto.service:auto-service:1.0-rc4'
  annotationProcessor 'com.google.auto.service:auto-service:1.0-rc4'
}

sourceCompatibility = "7"
targetCompatibility = "7"

```

##### TestRouterProcessor

这里一定要实现getSupportedAnnotationTypes()，否则不会触发process()

>  
 新手如果对注解的运行过程不是很了解，可以自己查下如何debug AbstractProcessor。操作并不复杂，本文也不多加阐述了。 


```
@AutoService(Processor.class)
public class TestRouterProcessor extends AbstractProcessor {<!-- -->

  public static final String ROOT_INIT = "com.example.generated";
  public static final String INIT_CLASS = "TestRouterInit";
  public static final String INIT_METHOD = "init";

  @Override public synchronized void init(ProcessingEnvironment processingEnvironment) {<!-- -->
    super.init(processingEnvironment);
  }

  //这个方法非常必要，否则将不会执行到process()方法
  @Override public Set&lt;String&gt; getSupportedAnnotationTypes() {<!-- -->
    return Collections.singleton(TestRouter.class.getCanonicalName());
  }

  @Override public SourceVersion getSupportedSourceVersion() {<!-- -->
    return SourceVersion.latestSupported();
  }

  @Override
  public boolean process(Set&lt;? extends TypeElement&gt; annotations, RoundEnvironment env) {<!-- -->
    System.out.println("TestRouterProcessor process");
    if (annotations == null || annotations.isEmpty()) {<!-- -->
      return false;
    }
    for (Element element : env.getElementsAnnotatedWith(TestRouter.class)) {<!-- -->
      //获取注解中的内容
      String className = element.getSimpleName().toString();
      String uri = element.getAnnotation(TestRouter.class).url();

      try {<!-- -->
        //使用javapoet来动态生成代码
        MethodSpec main = MethodSpec.methodBuilder(INIT_METHOD)
            .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
            .returns(void.class)
            .addStatement("$T.getInstance().register($S,$S)", RouterManager.class, className, uri)
            .build();

        TypeSpec testRouterInit = TypeSpec.classBuilder(INIT_CLASS)
            .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
            .addMethod(main)
            .build();

        JavaFile javaFile = JavaFile.builder(ROOT_INIT, testRouterInit)
            .build();

        Filer filer = processingEnv.getFiler();
        javaFile.writeTo(filer);
      } catch (Exception e) {<!-- -->
        e.printStackTrace();
      }
    }

    return true;
  }
}

```

# 使用注解

##### build.gradle（app模块）

主要引用了两个模块：
1. annotationrouter：引用注解模块，这样可以再Activity上使用TestRouter
1. processortest：使用注解解释器触发processortest模块，编译结束后可以动态生成TestRouterInit文件。

 <img src="https://img-blog.csdnimg.cn/20200322161121966.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" width="50%" height="50%">

```
apply plugin: 'com.android.application'

——中间代码省略，都是自动生成的——

dependencies {<!-- -->
  implementation 'androidx.appcompat:appcompat:1.1.0'
  implementation 'androidx.constraintlayout:constraintlayout:1.1.3'

  implementation project(':annotationrouter')
  annotationProcessor project(':processortest')
}

```

##### MainActivity

```
@TestRouter(url = "scheme://test")//使用注解
public class MainActivity extends AppCompatActivity {<!-- -->

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    RouterManager.getInstance().init();//初始化注解，init中调用的代码是动态生成的
    RouterManager.getInstance().showAllActivity();//show目前注册的
  }
}

```

### 运行成功后的log

```
com.example.annotaiontest I/System.out: RouterManager:{<!-- -->MainActivity=scheme://test}
```