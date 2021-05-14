#动态编译 java  ASM入门
# 概述

ASM 是java字节码操作框架。 由于ASM性能好的原因，所以在动态编译上往往比Javassist上使用的更加广泛。

之前已经写过了Javassist实现动态编译的demo，对动态编译不了解的读者可以看下：

本文在前面demo的基础上，将Javassist的实现改为了ASM。 因此对于gradle 插件等重复的点就不多加描述了，本文主要讲解下ASM的使用。

# Demo 概述

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911448650.png " width="40%" height="40%"><img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911450051.png " width="40%" height="40%"> 本demo通过ASM，实现在方法中动态插入代码的功能。 主要代码如下：

```
public class PluginTestClass {<!-- -->

  public void init() {<!-- -->
    System.out.println("PluginTestClass init");
    //此处将会使用动态编译插入代码
    //PluginTestClass.testPrint();
  }

  public static void testPrint(String s) {<!-- -->
    System.out.println(s);
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

### Demo结果

```
2020-04-05 10:26:53.708 11336-11336/com.example.transformtest I/System.out: PluginTestClass init
2020-04-05 10:26:53.708 11336-11336/com.example.transformtest I/System.out: 我是插入的代码

```

# “代码插入”实现流程

1、自己实现了TestFileUtils.java这个类来递归遍历文件夹下的所有文件。 2、遍历所有文件找到PluginTestClass这个类。 3、使用ClassVisitor 和MethodVisitor 分别来找到init方法和修改init方法。

>  
 MethodVisitor中的visitMaxs方法用于返回最大的操作数栈和局部变量表，这两项往往会根据插入的代码而改变。（此demo中插入的代码不会导致这两者改变，因此此demo中没有修改） 笔者自己单独写了一篇文章用于讲解着两个概念，对其有兴趣的读者可以看下：  


### TestTransform.java

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
          scanFilesAndInsertCode(src);
          FileUtils.copyDirectory(src, dst);
        } catch (Exception e) {<!-- -->
          System.out.println(e.getMessage());
        }
      });
    }
  }

  private void scanFilesAndInsertCode(File file) throws Exception {<!-- -->
    TestFileUtils.scanFileInDir(file,
        new TestFileUtils.ScanFileCallback() {<!-- -->
          @Override public void action(File file) {<!-- -->
            if (file.getAbsolutePath().contains("PluginTestClass")) {<!-- -->
              insertTestCode(file);
            }
          }
        });
  }

  private void insertTestCode(File file) {<!-- -->
    try {<!-- -->
      //读取class文件并且插入代码
      InputStream inputStream = new FileInputStream(file);
      ClassReader cr = new ClassReader(inputStream);
      ClassWriter cw = new ClassWriter(cr, 0);
      ClassVisitor cv = new MyClassVisitor(Opcodes.ASM5, cw);
      cr.accept(cv, ClassReader.EXPAND_FRAMES);

      //生成新的class文件
      FileOutputStream fileOutputStream = new FileOutputStream(file);
      fileOutputStream.write(cw.toByteArray());
      fileOutputStream.close();
    } catch (Exception e) {<!-- -->
      e.printStackTrace();
    }
  }

  private static class MyClassVisitor extends ClassVisitor {<!-- -->

    MyClassVisitor(int api, ClassVisitor cv) {<!-- -->
      super(api, cv);
    }

    public void visit(int version, int access, String name, String signature,
        String superName, String[] interfaces) {<!-- -->
      super.visit(version, access, name, signature, superName, interfaces);
    }

    @Override
    public MethodVisitor visitMethod(int access, String name, String desc,
        String signature, String[] exceptions) {<!-- -->
      MethodVisitor mv = super.visitMethod(access, name, desc, signature, exceptions);
      //generate code into this method
      if (name.equals("init")) {<!-- -->
        mv = new MyMethodVisitor(Opcodes.ASM5, mv);
      }
      return mv;
    }
  }

  private static class MyMethodVisitor extends MethodVisitor {<!-- -->

    MyMethodVisitor(int api, MethodVisitor mv) {<!-- -->
      super(api, mv);
    }

    @Override
    public void visitInsn(int opcode) {<!-- -->
      if ((opcode &gt;= Opcodes.IRETURN &amp;&amp; opcode &lt;= Opcodes.RETURN)) {<!-- -->
        //添加方法
        mv.visitLdcInsn("我是插入的代码");
        mv.visitMethodInsn(Opcodes.INVOKESTATIC, "com/example/testplugin/PluginTestClass",
            "testPrint",
            "(Ljava/lang/String;)V", false);
      }
      super.visitInsn(opcode);
    }

    @Override public void visitMaxs(int maxStack, int maxLocals) {<!-- -->
      super.visitMaxs(maxStack, maxLocals);
    }
  }
}


```

### TestFileUtils.java

```
public class TestFileUtils {<!-- -->
  public static void scanFileInDir(File rootFile, ScanFileCallback callback) {<!-- -->
    if (rootFile == null) {<!-- -->
      return;
    }
    if (rootFile.isDirectory()) {<!-- -->
      if (rootFile.listFiles() == null) {<!-- -->
        return;
      }
      for (File file : rootFile.listFiles()) {<!-- -->
        scanFileInDir(file, callback);
      }
    } else {<!-- -->
      callback.action(rootFile);
    }
  }

  public interface ScanFileCallback {<!-- -->
    void action(File file);
  }
}

```