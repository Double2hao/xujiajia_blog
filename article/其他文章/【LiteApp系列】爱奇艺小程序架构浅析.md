#【LiteApp系列】爱奇艺小程序架构浅析
# 前言

上一篇文章已经讲述了何为小程序，地址如下：

>  
  


此篇主要讲一下其架构设计。

# 对WebView的优化

传统的WebView使用过程如下：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911169250.png " alt="这里写图片描述" title="">  从图中可以看到，传统的WebView在使用的时候，会有较长时间处于无法操作的状态。（不一定是白屏，有的可以使用引导图之类）

小程序主要从以下几个方面进行了优化：  1、在APP或者说进程启动的时候就在后台做完WebView的初始化工作，这样在使用的时候就可以节省等待初始化的时间。  2、添加了对网络框架的缓存功能，这样在第一次之后加载的时候就不需要重新到网络去下载，直接本地读取就可以了。  3、使用vue.js框架，将VDom操作与渲染分离。在android native提前启动JavaScriptCore用来负责VDom的构建和找Diff等工作，而WebView只需要负责渲染即可。这样在WebView加载其他JS代码的时候，JavaScriptCore就可以开始构建VDom。

优化之后的WebView的使用过程如下：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911170661.png " alt="这里写图片描述" title="">

# 系统架构

下图摘自github：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911171512.png " alt="这里写图片描述" title="">  上图摘自官方github开源项目，主要有以下几个要点：  1、JS代码主要有两个框架JSBridger和VUE。VUE框架负责主要的业务逻辑和界面，JSBridger主要负责与android native的交互。  2、JS运行在android native上，负责VDOM Diff相关工作。WebView则主要只负责渲染。  3、对Webview或者是native UI的事件触发开源通过JsBridger来与JS进行交互，再而更新WebView或者是native UI。

>  
 更多内容可以查看github官方开源项目：   
