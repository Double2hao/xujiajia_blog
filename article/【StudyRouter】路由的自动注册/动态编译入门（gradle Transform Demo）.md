#动态编译入门（gradle Transform Demo）
# 概述

现在市面上的插件化框架，热修复框架几乎都使用了动态编译技术。

动态编译的实质是，使用gradle transform api，在项目构建过程的class文件转成dex文件之前，通过自定义插件，进行class字节码处理。

>  
 本文主要是通过走一遍简单Demo实现流程，让读者能对动态编译有一个大概的了解。 如对一些细节知识有更多需求的读者就需要自行学习了。 


# 简单Demo

本文的Demo，通过动态编译实现在代码中插入一行代码。 主要实现步骤如下：
1. 实现gradle Plugin。1. 实现Transform，并且在Plugin中注册。1. Plugin编译，并且上传到本地仓库。1. app项目应用Plugin，通过插件实现动态编译。1. 运行项目，查看动态编译结果。
# Demo结果

```
public class PluginTestClass {<!-- -->

  public void init(){<!-- -->
    System.out.println("PluginTestClass init");
    //这里将会插入System.out.println("我是插入的代码");
  }
}

```

```
public class MainActivity extends AppCompatActivity {<!-- -->
  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    PluginTestClass pluginTestClass=new PluginTestClass();
    pluginTestClass.init();
  }
}

```

执行结果：

```
2020-03-29 09:04:37.029 26361-26361/? I/System.out: PluginTestClass init
2020-03-29 09:04:37.029 26361-26361/? I/System.out: 我是插入的代码

```

# testplugin模块

这个模块主要实现了两个内容：
1. 实现插件1. 实现Transform，编辑class文件，插入代码。
##### build.gradle

```
apply plugin: 'groovy'
apply plugin: 'maven'
apply plugin: 'java'

dependencies {
  compile gradleApi()//gradle sdk
  compile localGroovy()//groovy sdk

  implementation 'com.android.tools.build:gradle:3.6.1'
  implementation 'org.javassist:javassist:3.27.0-GA'//用于编辑class文件
}

repositories {
  mavenCentral()
}

//提交仓库到本地目录
def version = "1.0.0";
def artifactId = "testplugin";
def groupId = "com.example.plugin.test";
uploadArchives {
  repositories {
    mavenDeployer {
      repository(url: uri('./repo')) {
        pom.groupId = groupId
        pom.artifactId = artifactId
        pom.version = version
      }
    }
  }
}


```

##### TestPlugin.java

```
public class TestPlugin implements Plugin&lt;Project&gt; {<!-- -->

  @Override
  public void apply(Project project) {<!-- -->
    System.out.println("这是自定义插件!");
    project.getExtensions().findByType(BaseExtension.class)
        .registerTransform(new TestTransform());
  }
}

```

##### TestTransform.java

```
public class TestTransform extends Transform {<!-- -->

  //用于指明本Transform的名字，也是代表该Transform的task的名字
  @Override public String getName() {<!-- -->
    return "TestTransform";
  }

  //用于指明Transform的输入类型，可以作为输入过滤的手段。
  @Override public Set&lt;QualifiedContent.ContentType&gt; getInputTypes() {<!-- -->
    return TransformManager.CONTENT_CLASS;
  }

  //用于指明Transform的作用域
  @Override public Set&lt;? super QualifiedContent.Scope&gt; getScopes() {<!-- -->
    return TransformManager.SCOPE_FULL_PROJECT;
  }

  //是否增量编译
  @Override public boolean isIncremental() {<!-- -->
    return false;
  }

  @Override public void transform(TransformInvocation invocation) {<!-- -->
    System.out.println("TestTransform transform");
    for (TransformInput input : invocation.getInputs()) {<!-- -->
      //遍历jar文件 对jar不操作，但是要输出到out路径
      input.getJarInputs().parallelStream().forEach(jarInput -&gt; {<!-- -->
        File src = jarInput.getFile();
        System.out.println("input.getJarInputs fielName:" + src.getName());
        File dst = invocation.getOutputProvider().getContentLocation(
            jarInput.getName(), jarInput.getContentTypes(), jarInput.getScopes(),
            Format.JAR);
        try {<!-- -->
          FileUtils.copyFile(src, dst);
        } catch (IOException e) {<!-- -->
          throw new RuntimeException(e);
        }
      });
      //遍历文件，在遍历过程中
      input.getDirectoryInputs().parallelStream().forEach(directoryInput -&gt; {<!-- -->
        File src = directoryInput.getFile();
        System.out.println("input.getDirectoryInputs fielName:" + src.getName());
        File dst = invocation.getOutputProvider().getContentLocation(
            directoryInput.getName(), directoryInput.getContentTypes(),
            directoryInput.getScopes(), Format.DIRECTORY);
        try {<!-- -->
          scanFilesAndInsertCode(src.getAbsolutePath());
          FileUtils.copyDirectory(src, dst);
        } catch (Exception e) {<!-- -->
          System.out.println(e.getMessage());
        }
      });
    }
  }

  private void scanFilesAndInsertCode(String path) throws Exception {<!-- -->
    ClassPool classPool = ClassPool.getDefault();
    classPool.appendClassPath(path);//将当前路径加入类池,不然找不到这个类
    CtClass ctClass = classPool.getCtClass("com.example.testplugin.PluginTestClass");
    if (ctClass == null) {<!-- -->
      return;
    }
    if (ctClass.isFrozen()) {<!-- -->
      ctClass.defrost();
    }
    CtMethod ctMethod = ctClass.getDeclaredMethod("init");

    String insetStr = "System.out.println(\"我是插入的代码\");";
    ctMethod.insertAfter(insetStr);//在方法末尾插入代码
    ctClass.writeFile(path);
    ctClass.detach();//释放
  }
}

```

##### TestPlugin.properties

这个文件注意，一定要在这个目录下，否则会找不到插件。

```
implementation-class=com.example.testplugin.TestPlugin

```

##### 使用gradle将上传到私有仓库

由于在build.gradle中配置的仓库地址是"./repo"。 最终执行结束后，项目中可以看到这个仓库：

# app模块

这个模块主要实现两个内容
1. 应用插件1. 实现demo代码
##### TransformTest/build.gradle

需要再根目录添加插件的仓库地址和依赖插件。

>  
 笔者在testplugin项目中生成repo仓库后，会再复制一份到根目录，这份根目录的repo才是真正使用到的仓库。 代码多一个copy的过程的目的：主要是避免testplugin/repo仓库删除的时候项目没法编译。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039204310.png" width="30%" height="30%"> 


```

buildscript {<!-- -->
    
    repositories {<!-- -->
        google()
        jcenter()
        maven {<!-- -->
            url uri('./repo')//添加依赖仓库
        }
    }
    dependencies {<!-- -->
        classpath 'com.android.tools.build:gradle:3.6.1'
        classpath 'com.example.plugin.test:testplugin:1.0.0'//依赖插件项目
    }
}

allprojects {<!-- -->
    repositories {<!-- -->
        google()
        jcenter()
        
    }
}

task clean(type: Delete) {<!-- -->
    delete rootProject.buildDir
}


```

##### TransformTest/app/build.gradle

```
apply plugin: 'com.android.application'
apply plugin: 'TestPlugin'//应用插件

——后面自动生成的代码省略——


```

##### PluginTestClass.java

```
public class PluginTestClass {<!-- -->

  public void init(){<!-- -->
    System.out.println("PluginTestClass init");
    //这里将会插入System.out.println("我是插入的代码");
  }
}


```

##### MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->
  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    PluginTestClass pluginTestClass=new PluginTestClass();
    pluginTestClass.init();
  }
}

```