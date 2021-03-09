#android Activity的启动模式 作用简析+demo详解
笔者近期做的一个项目用到了Activity的启动模式，也算是第一次深刻地领会到了其强大与方便。在此也是将自己所得与大家分享，自己写了一个比较简易的demo，便于让大家理解。

此篇博客意在让对启动模式不了解的开发者对其有一个较为形象的认识，至于深入探究，笔者还是推荐去看任玉刚前辈所写的《android开发艺术探索》了。

网上对Activity的启动模式讲解的博客有很多，但是大部分都需要掌握“栈”的知识，而且很多并不是那么通俗易懂。笔者打算独辟蹊径，**一方面通过百度地图讲其作用**，**另一方面通过自己写的demo演示来讲解4种启动模式**。

 

**作用：**

大家对android百度地图一定非常熟悉，让我们来看一下下面的图片：（百度地图）

  <img alt="" class="has" height="450" src="https://img-blog.csdn.net/20160319091825859?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="250">  <img alt="" class="has" height="450" src="https://img-blog.csdn.net/20160319091738328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="250">  <img alt="" class="has" height="450" src="https://img-blog.csdn.net/20160319091805906?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="250">  

假设以上3张图片依次为ABC，笔者得到这三张图的顺序分别为：（好奇的读者可以自己试试）

打开A，打开B，打开C，打开B，关闭B，关闭C，回到A，退出程序。（关闭均使用退回键）

 

大家可以仔细观察下A与C两张图，不难发现**这其实就是同一个Activity**，只是通过动态改变布局使得大家没有察觉到。

 

那么问题来了,是如何办到复用同一个Activity的呢？

无疑就是巧妙地使用了Activity的启动模式。

按照正常的启动模式，A打开B，B再打开C，应当会有3个Activity，但是很显然，我们这个test中只有2个Activity，证据就是处在C图片的时候按退回键并没有跳转到另外的一个Activity，而是改变了一下布局，随后再按退回键就会推出程序。

这是一个多么实用的技能啊！如果会了这个，我们就再也不用老是各种无脑的finish()了！

没错，这就是Activity的启动模式的一个作用了。

（至于是否有其他什么作用，笔者也还未领悟，欢迎指点）

 

 

**demo讲解：**

首先我们是要知道一共有四种启动模式：standard（标准模式），singleTop（栈顶复用模式），singleTask（栈内复用模式），singleInstance（单例模式）。

 

新手可能会询问启动模式在哪里设置，笔者在这里解答一下：

在AndroidManifest里面，Activity的launchMode属性中直接可以设置。

 

 

**standard:标准模式，这也是系统的默认模式。**

每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否存在。

<img alt="" class="has" height="450" src="https://img-blog.csdn.net/20160319094813183?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300">

如图，笔者依次打开的Activity是，red,blue,green,green。然后按退回键关闭的顺序分别为，green，green，blue，red。

 

 

**singleTop：栈顶复用模式。**

其实解释很简单，像上面的standard，我们可以发现，green的Activity是可以再启动一个green的Activity的，他居然可以自己跳转到自己，简直太荒唐了，浪费内存。设置了singleTop之后，如果还有“自己跳转自己”的操作，就不会再创建一个新的Activity了。

<img alt="" class="has" height="450" src="https://img-blog.csdn.net/20160319095617534?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300">

如图，笔者red的Activity设置了singleTop的启动模式。

从blue的Activity跳转到了red的Activity，但是当red自己跳转到自己的时候，就不会再次创建一个新的实例。

 

 

**singleTask：栈内复用模式。**

这个启动模式就更方便了，更像是笔者上面所讲述的百度地图的地图界面的启动模式。在这种模式下，如果这个Activity已经被创建过了，那么就不会再次被创建了，而是将之前创建过的那个实例拿过来直接用。

但是如果Activity没有创建过，那么就会重新创建一个任务栈，并把新创建的Activity放入。需要理解这个就必须要懂栈的概念了，笔者此处为了让新手理解，先避开。

<img alt="" class="has" height="500" src="https://img-blog.csdn.net/20160319101113225?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300">

如图，笔者blue的Activity设置了singleTask的启动模式。

笔者依次打的Activity为，red，blue，green，red,blue。细心的同学已经发现，当最后按返回键的时候，直接返回到red之后就退出了Activity，说明只剩下red一个Activity了，这是为何呢？

主要有三点原因：（不懂栈的先只需要理解第一个即可，如有兴趣，可以自己再深入理解）

1、第二个blue复用了之前的一个blue

2、创建第一个blue的时候新建了一个任务栈

3、因为singleTask默认具有clearTop的效果，比如是ADBC 4个Activity,倘若D为singleTask，那么当再次启动D的时候，就会只留下AD两个Activity，中间了Activity都被clear了。

此处两个blue之间的green和red都被clear了。所以打开第二个blue的时候实际上存在两个任务栈，第一个任务栈只有一个red，第二个任务栈只有一个blue，所以按返回的时候就回到了red，再按返回就会退出了。

 

 

 

**singleInstance：单例模式。**

这个模式如果不用“栈”的思想来讲真的比较复杂。但是倘若连“栈”都不懂，那么一般也用不到这儿启动模式了。

稍微讲解一下：在默认的启动模式下，当我们启动Activity的时候，系统会创建多个实例并把他们一一放入任务栈，当我们按back键，这些Activity就会一一退回。但是在singleInstance中就有所不同了，singleInstance的Activity是直接新建一个任务栈，并且独自运行在里面，并且由于栈内复用性的特性，后序均不会创建新的Activity。

<img alt="" class="has" height="450" src="https://img-blog.csdn.net/20160319102931404?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="300">

 

 

笔者此篇博客意在让新手更能理解Activity的启动模式，可能有多处讲地不够严谨，如有前辈不吝指点，不甚感激。

 

示范demo下载地址：
