#快速理解android事件传递拦截机制概念
listview与scollview嵌套使用过的小伙伴一定烦心过其滑动事件的麻烦。

打个比方：

假如有3个布局，Scollview A ，Scollview B，ListView C，B是A的子部局，C是B的子部局。ABC三者都是垂直滑动，那么当我触摸手机向下滑动的时候，滑动的是哪个view呢？

倘若明白了android事件分发机制，这些就很容易理解了。 

 

以下为部分原理：（经常碰到的方法是“**事件拦截**”和“**事件响应**”）

ViewGroup中的三个方法：

事件分发(dispatchTouchEvent(MotionEvent ev))

事件拦截(onInterceptTouchEvent(MotionEvent ev))

事件响应(onTouchEvent(MotionEvent ev)) 

View中只有两个方法：

事件分发(dispatchTouchEvent(MotionEvent ev)) 

事件响应(onTouchEvent(MotionEvent ev)) 

 

三个方法的调用流程大致如下：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/940.png" alt=""> 

 

看到几个函数眼花缭乱？**完全没关系！**笔者将用**更通俗易懂**的方式解释一遍。

我们还是举一个例子：

LinearLayout A ，Scollview B，ListView C 

B是A的子部局，C是B的子部局 

 

当发生触摸事件，事件的分发次序如下：A-&gt;B-&gt;C

当分发事件结束，事件开始处理，次序如下：C-&gt;B-&gt;A

 

在《Android群英传》中有比较形象的比喻，在此引用一下，希望帮助大家理解：

假设A为总经理，B为部长，C为员工。

一旦有事情，A会通知B，然后B通知C。

事件分发完毕后，从C开始执行。C做完了自己的事情，就通知B，然后B通知A。

 

那么再讨论一种情况，倘若我只想要第一个A获取到点击事件，而B和C不用滑动呢？

A直接可以在事件分发的时候就 不告诉B和C有这个事件的发生。

也就是最终是：A接收，A不分发事件，A处理事件。

 

那么倘若A不仅仅自己要可以获取到触摸事件，而且还要B可以滚动，但是却不想要C和B造成滑动冲突呢？

这次就可以让B不告诉C事件的发生。

最终事件分发次序如下：A-&gt;B

事件处理次序如下：B-&gt;A

 

当然还有一种与上述两种完全不同的情况，我们不希望A的触摸事件执行，但是B和C却需要获取到触摸事件。

这次就可以让B在处理事件之后，不告诉A。



最终事件分发次序如下：A-&gt;B-&gt;C

事件处理次序如下：C-&gt;B

 

此篇文章更注重概念的理解，如果读者有兴趣，可以再查看以下两篇文章。

 

此篇文章使用log打印的方式解释。

 

此篇文章从源码角度解释。 

 

 