#【LiteApp系列】何为爱奇艺小程序？
# 前言

今年5月份，爱奇艺开源了LiteApp，俗称小程序。LiteApp是一种是一种高性能的移动跨平台框架，它集合了Native App和Web App的优势。Github地址如下：

>  
  


那么，既然已经有了Native App和Web App，为何还要有LiteApp呢，它解决了什么问题呢？

# Native App

Native App就是Android或者ios原生开发的App。  Native 毫无疑问是官方最推荐的开发方式，使用原生开发，不仅仅是硬件上的支持，软件开发文档也是十分完善。

## Native App弊端

1、不能跨平台。android和ios两个平台需要有两套不同的代码。  2、没有比较完善的动态化。如果功能上有一些的改动，需要在应用市场重新上架。另外，纯 Native 动态加载，在 Android 和 IOS 商店中不允许上架。  3、发布上线周期长。由于需要同时开发两套代码，开发上就有不同的问题，同时也要涉及到两个市场的不同问题，因此开发上线周期会比较长。

# Web App

Web App很好地解决了Native App不能跨平台，没有比较完善的动态化的问题。  Web App在App端使用WebView来加载Web，主要的业务开发在Web端，这样不仅能让android和ios端共用一套代码，而且只需要更改Web端的代码，App端即使不在应用商店更新的情况下，也可以更新应用功能。

## Web App弊端

1、打开速度慢。WebView 启动时需要进行加载浏览器内核等操作。  2、操作不流畅，卡顿。WebView 业务逻辑运行和网页渲染是串行的，较重的业务逻辑会造成卡顿。另外，WebView 没有比较好的缓存机制，前端资源需要每次去服务器下载。  3、没有权限机制。与第三方联合开发时无法防御来自第三方的攻击。

# LiteApp

LiteApp集合了Native App和Web App的优势，使用WebView+Native的开发方式。开发人员不仅可以使用现代Web开发技术，同一套代码在android和ios上使用，而且还让性能更加接近于原生，有比较好的缓存机制，权限机制等。

主要特点如下：  1、高性能。在Web上编写，具有与本机应用程序相同的性能  2、移动跨平台。使用单一代码库构建Android和iOS  3、启动快，不卡顿。浏览器渲染与业务逻辑线程分离。  4、为所有页面加载快速快速渲染，尤其是第一次。  5、可扩展的 专有API用于扩展，它可以添加更多功能。

# Web App与LiteApp性能对比

<th align="left">项目</th><th align="left">加载时间/毫秒</th>|切换页面/ fps
|------
<td align="left">LiteApp</td><td align="left">250-500毫秒</td>|完美的/ 60
<td align="left">HTML5应用程序</td><td align="left">大于1000ms</td>|白色屏幕短时间/ 53

## Web App

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210038876660.png" alt="这里写图片描述" title="">

## LiteApp

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210038978901.png" alt="这里写图片描述" title="">

>  
 更多内容可以查看github官方开源项目：   
