#android studio sdk更新 （2015-8-5）
最近也是自己遇到了sdk更新的问题，由于部分的资料都比较老，无法解决新的问题，我在网上搜索也是花了一定的时间，也是将更新的方法稍微整理一下，供大家参考。

 

首先要修改系统文件hosts,如图：

<img alt="技术分享" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911710280.png ">

路径基本都差不多，自己根据自己的系统看咯

接着用工具比如文本查看器等打开这个文件，如果没有权限，则想办法添加可编辑这个文件的权限，比如右键属性添加操作权限，

打开如下图：

<img alt="技术分享" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911712511.png ">

然后添加这2句代码到末尾

 

203.208.46.146 dl.google.com

203.208.46.146 dl-ssl.google.com

添加好后如下图：

 

<img alt="技术分享" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911713392.png ">

 

接下来打开SDK里面的Android SDK Manager，在Tools下的 Options 里面，有一项 Force https://..sources to be fetched using http://... 将这一项勾选上，添上对应的HTTP地址和端口，如下图：

<img alt="技术分享" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911715393.png ">

 