#android官方Api 理解Activity生命周期的回调机制（适合有基础的人看）
原文地址：

 

此处笔者看到，主要是“android替代main函数启动方式”的概念让笔者有豁然开朗的感觉，之前也很少会去思考这种问题，翻译一下，希望能够让更多的与笔者一样未了解此点的人一点帮助。

第一次翻译，如有意见，欢迎评论交流。

 

 

启动一个Activity

不像其他应用的编程范例一样使用main函数的启动方式，android系统通过唤醒具体的与生命周期阶段相关的回调方法来启动一个Activity实例。有一个序列的回调方法来启动一个Activity和摧毁一个Activity。

这节课提供了一个最重要的生命周期的概观图，并且告诉你如何处理第一个新建的Activity实例的生命周期的回调。

 

理解生命周期的回调机制

在一个Activity的生命中，这个系统在一个阶梯金字塔的序列下调用了一组核心的生命周期方法。意思就是，每一个生命周期的阶段都对应着一个在金字塔中单独的阶梯。这个系统打开一个新的Activity实例的同时，每一个回调方法把Activity的状态一步步移向顶端。这金字塔的顶端是Activity在前台运行的关键，并且只有这样使用者们才能与其互动。

在使用者开始离开Activity的时候，这个系统调为了干掉这个Activity，用了其他的方法来让Activity的状态一步步移向底端。在一些情况下，Activity不会完全跌到金字塔的底端，他会下来一部分并且等待（比如使用者跳转到另一个APP的时候），通过这种方式，Activity能够再回到顶端（如果使用者再回到了这个Activity），并且重新占用离开的时候使用的东西。

 









<img alt="" class="has" src="https://img-blog.csdn.net/20160324083006698?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 