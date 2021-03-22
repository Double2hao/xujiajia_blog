#android 取消标题栏出错，程序崩溃的问题
在android studio中使用取消标题栏的时候偶尔会出现设置之后程序崩溃的问题，由于自己也处于学习时期，所以之前大多复制一下其他人的代码，然后就跳过如此问题，近期实在想搞懂这个，于是稍微花时间看了下代码。

 

先看一下错误的代码，如此运行程序会崩溃：

<img alt="" class="has" src="https://img-blog.csdn.net/20151011175837992?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

大家有没有主要到什么问题——为什么是 AppcompatActivity而不是Activity呢？

换成Activity试试：

<img alt="" class="has" src="https://img-blog.csdn.net/20151011180011820?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

如此，换了之后程序便可以运行了。

那AppcompatActivity到底是什么呢，和Activity又有什么区别？就由好奇的同学自己去看下源码或者百度了。

 