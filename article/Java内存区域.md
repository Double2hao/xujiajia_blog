#Java内存区域
# 概要

近期看知乎脉脉上其他人对Java知识的讲解，发现自己居然连Java内存区域的各种内容都记不太清了。遍历博客文章的时候发现也没有这方面的记载，好像都记录在毕业前面试小本子上了。 之后可能会开始捡起一写Java基础知识，也是温故而知新吧。

>  
 本文参考《深入理解Java虚拟机》 


# Java内存区域图

图片摘自《深入理解Java虚拟机》 <img src="https://img-blog.csdnimg.cn/20190629153932373.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> Java内存主要区域：
- 程序计数器- 虚拟机栈- 本地方法栈- 堆- 方法区（包括常量池）
## 程序计数器

线程私有，每条线程有一个。 字节码解释器通过改变程序计数器来选取下一条需要执行的字节码指令。指令包括：分支、循环、跳转、异常处理。

## 虚拟机栈

线程私有，每条线程有一个。 存Java对象的引用。

## 本地方法栈

线程私有，每条线程有一个。 存native对象的引用。

## Java堆

所有线程共享。 存放对象实例。（不包括静态对象）

## 方法区

（在 1.7 版本的 HotSpot 虚拟机中，方法区被称为永久代Permanent Space，而在 JDK 1.8 中则被称之为 元空间MetaSpace） 所有线程共享。 存储类信息、常量、静态变量、运行时编译的代码等数据。 方法区中的常量存储在“常量池”中。
