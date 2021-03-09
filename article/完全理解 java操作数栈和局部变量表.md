#完全理解 java操作数栈和局部变量表
# 概要

近期学习到字节码操控框架ASM，其中对方法的定义需要设置最大操作数栈和局部变量表。 于是，自己又复习了一遍java栈帧的概念。

>  
 如果对栈帧概念还不了解的读者推荐看下此文章： 


# 例子综述

本文将会通过诸多字节码的例子，来具体分析在不同情况下的方法的操作数栈和局部变量表。

>  
 本文的分析主要基于javac和javap的使用: 
 - 首先用javac生成java文件编译后的class文件- 再使用"javap -v xxx.class"来输出该class文件的信息 


#### 操作数栈可以复用

首先要强调个概念：“操作数栈可以复用”。如下两个例子中test方法的操作数栈都会是2。

```
public class Test {<!-- -->
  public int test(int a, int b) {<!-- -->
    int c = a + b;
    return c;
  }
}

```

```
public class Test {<!-- -->
  public int test(int a, int b) {<!-- -->
    int c = a + b;
    c = a + b;
    c = a + b;
    c = a + b;
    return c;
  }
}

```

### 1、对象方法：方法参数
- 操作数栈：2 先load a，再load b- 局部变量表：3 this，a，b
```
public class Test {<!-- -->
  public int test(int a, int b) {<!-- -->
    return a + b;
  }
}

```

```
public int test(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: iload_1
         1: iload_2
         2: iadd
         3: ireturn
      LineNumberTable:
        line 10: 0

```

### 2、对象方法：本地变量+方法参数
- 操作数栈：2 先load a，再load b- 局部变量表：2 this，b
```
public class Test {<!-- -->
  private int a = 0;
  public int test(int b) {<!-- -->
    return a + b;
  }
}

```

```
  public int test(int);
    descriptor: (I)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: getfield      #2                  // Field a:I
         4: iload_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 11: 0

```

### 3、对象方法：方法变量+方法参数
- 操作数栈：2 先load a，再load b- 局部变量表：4 this，a，b，c
```
public class Test {<!-- -->
  public int test(int a, int b) {<!-- -->
    int c = a + b;
    return a + b;
  }
}

```

```
  public int test(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=3
         0: iload_1
         1: iload_2
         2: iadd
         3: istore_3
         4: iload_1
         5: iload_2
         6: iadd
         7: ireturn
      LineNumberTable:
        line 10: 0
        line 11: 4

```

### 4、静态方法：方法参数
- 操作数栈：2 先load a，再load b- 局部变量表：2 a，b
```
public class Test {<!-- -->
  public static int test(int a, int b) {<!-- -->
    return a + b;
  }
}

```

```
  public static int test(int, int);
    descriptor: (II)I
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=2
         0: iload_0
         1: iload_1
         2: iadd
         3: ireturn
      LineNumberTable:
        line 10: 0

```

### 5、静态方法调用静态方法
- 操作数栈：1 将string“123” 压入栈中- 局部变量表：0
```
public class Test {<!-- -->
  public static int test() {<!-- -->
    add("123");
    return 0;
  }

  public static int add(String s) {<!-- -->
  }
}

```

```
  public static int test();
    descriptor: ()I
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: ldc           #2                  // String 123
         2: invokestatic  #3                  // Method add:(Ljava/lang/String;)V
         5: iconst_0
         6: ireturn
      LineNumberTable:
        line 10: 0
        line 11: 5

```

### 6、静态方法调用对象方法
- 操作数栈：2 将Test对象压入栈中，将string“123” 压入栈中- 局部变量表：0
```
public class Test {<!-- -->
  public static int test() {<!-- -->
    new Test().add("123");
    return 0;
  }

  public void add(String s) {<!-- -->
  }
}

```

```
  public static int test();
    descriptor: ()I
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: new           #2                  // class Test
         3: dup
         4: invokespecial #3                  // Method "&lt;init&gt;":()V
         7: ldc           #4                  // String 123
         9: invokevirtual #5                  // Method add:(Ljava/lang/String;)V
        12: iconst_0
        13: ireturn
      LineNumberTable:
        line 10: 0
        line 11: 12

```
