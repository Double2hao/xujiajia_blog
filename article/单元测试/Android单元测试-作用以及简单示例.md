#Android单元测试-作用以及简单示例
# 前提概要

## 受人嫌弃的单元测试

对于单元测试这个知识点，其实很多开发者是不太接触的，包括笔者，在实习之前也并未实用过单元测试，或者说并没感受到单元测试的好处。  对于bug的调试，笔者之前更倾向于使用log和断点调试，可以说会了这两个，大部分的逻辑bug都能自己解决了。这两个与看似臃肿的单元测试代码相比更受大家的喜爱。  但是，使用log和断点调试的前提是开发人员较少，甚至是单人开发的情况。如果我自己开发，我完全可以每次都使用集成测试，我知道每一个功能会涉及哪些模块的代码，然后根据逻辑设置log或者断点调试。

## 多人开发难以处理的问题

然而，如果是多人开发呢？每一个模块的代码很可能是由不同的人分开负责的，bug的产生由不同模块共同产生。每一个模块的代码可能都比较复杂，产生bug后，阅读其他人的模块本身比较浪费时间，其次基本不可能让你去修改其他人的代码，这可能会破坏他人的代码结构。  而且错误可能也并不在其他人的代码中，也可能是你们的交互方式有问题。产生bug的原因有太多，并且由单人直接log或者断点调试难以处理，那么这种情况怎么办呢？

单元测试就一定程度上处理了这种困难的情况：给每一个模块加上单元测试，如果该模块可以通过单元测试，就代表没有问题。  在这种情况下，程序员们面对的问题不再是要让整个项目到达理想的效果，而是让自己所面对的单元测试可以通过。这样就大大减少了多人开发中的交互成本。

# 简单示例

主要就两个文件：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040253020.png" alt="这里写图片描述" title="">

```
package com.example.xujiajia_sx.myexpressotest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {<!-- -->

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

    }

    public static int  calculate(int a,int b){
        return a+b+1;
    }

}

```

```
package com.example.xujiajia_sx.myexpressotest;

import org.junit.Test;

import static org.junit.Assert.*;

/**
 * Created by xujiajia_sx on 2017/8/14.
 */

public class SimpleTest {<!-- -->

    @Test
    public void CalculateTest() throws Exception {
        assertEquals(4, MainActivity.calculate(1,2));
    }
}

```

这个例子是测试了MainActivity.calculate()方法。可以在不运行这个app的情况下直接通过按SimpleTest .CalculateTest（）左边的小三角测试，如下图：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040253591.png" alt="这里写图片描述" title="">

我们calculate()方法的逻辑是返回a+b+1，所以是4，最终不会报错，如果我们把assertEquals中的4改成3，效果如下：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040254442.png" alt="这里写图片描述" title="">

如图，测试会直接报错，并且定位到错误的那一行，然后我们就可以看到是MainActivity.calculate()输出的值不等于3所造成的。

# Assert方法

示例本身比较简单，但是对于刚刚接触单元测试读者可能对assertEquals()比较陌生，这是Assert这个类中的静态方法，单元测试中一般就是通过它来判断是否达到理想的效果。  笔者此处使用了int之间的判断，Assert中还有很多其他的用法，笔者可以去AndroidDevelpers上自己查看，此处为了方便，笔者为了方便就直接复制了。

```
static void assertEquals(boolean expected, boolean actual)

static void assertEquals(String message, long expected, long actual)

static void assertEquals(short expected, short actual)

static void assertEquals(String message, String expected, String actual)

static void assertEquals(String message, int expected, int actual)

static void assertEquals(Object expected, Object actual)

static void assertEquals(String message, boolean expected, boolean actual)

static void assertEquals(String expected, String actual)

static void assertEquals(String message, short expected, short actual)

static void assertEquals(String message, Object expected, Object actual)

static void assertEquals(char expected, char actual)

static void assertEquals(byte expected, byte actual)

static void assertEquals(double expected, double actual, double delta)

static void assertEquals(String message, char expected, char actual)

static void assertEquals(float expected, float actual, float delta)

static void assertEquals(String message, double expected, double actual, double delta)

static void assertEquals(String message, byte expected, byte actual)

static void assertEquals(String message, float expected, float actual, float delta)

static void assertEquals(long expected, long actual)

static void assertEquals(int expected, int actual)

static void assertFalse(String message, boolean condition)

static void assertFalse(boolean condition)

static void assertNotNull(Object object)

static void assertNotNull(String message, Object object)

static void assertNotSame(String message, Object expected, Object actual)

static void assertNotSame(Object expected, Object actual)

static void assertNull(String message, Object object)

static void assertNull(Object object)

static void assertSame(String message, Object expected, Object actual)

static void assertSame(Object expected, Object actual)

static void assertTrue(String message, boolean condition)

static void assertTrue(boolean condition)

static void fail(String message)

static void fail()

static void failNotEquals(String message, Object expected, Object actual)

static void failNotSame(String message, Object expected, Object actual)

static void failSame(String message)

static String   format(String message, Object expected, Object actual)
```

# 总结

这篇文章主要介绍了Android单元测试的作用和简单的示例。但是简单的对方法的测试相信并不能满足求知欲强烈的读者。

下一篇文章笔者会讲述Android单元测试中对Activity的测试方法。