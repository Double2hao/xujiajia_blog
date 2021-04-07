# github项目地址
[https://github.com/Double2hao/StudyWebview](https://github.com/Double2hao/StudyWebview)

# StudyWebView架构

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/70.png" >

StudyRouter架构如上图，主要几个模块的功能如下：
- webview
客户端的webview，可以是view，也可以是界面。H5运行于webview上，webview是H5与客户端交互的媒介。
- Bridges
Bridges处于业务进程。H5通过单一的通道来调用客户端的功能，
由于客户端部分业务会处于业务进程，因此webview会将需要调用业务进程的逻辑封装成bridges。
功能例如：调用文件下载，检查登录状态，获取用户信息等。
- Component
Component处于webview进程。H5通过单一的通道来调用客户端的功能。
一方面，部分基础功能不需要经历IPC（进程间通信），另一方面webview进程扔可能存在需要在本进程处理的客户端逻辑。
功能例如：启动其他页面，弹窗提醒，重启页面等。

> Components相关功能完全可以由webview所处的页面来处理，是不是没有必要独立搞个模块？
> **Components存在的必要性**：
> 1. 如果业务都写在webview所在页面，会导致页面的逻辑不断复杂，在项目不断发展的同时，webview的页面会变得越发的臃肿。
> 2. 使用Components能让各块功能之间互相解耦，提高了拓展性和可维护性。

# 客户端webview一般原理

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/71.png" >


## webview与其他业务同一个进程
客户端与H5的通信是双向的：
- 客户端->H5
直接通过webview执行js代码
- H5->客户端
通过webview通知到客户端，然后客户端根据cmd找到对应的bridge，执行相关逻辑。

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/72.png" >


## webview单独进程
webview一般建议设置单独进程。主要是为了避免影响业务进程，一旦有问题，尽量只让webview进程崩溃，而不影响业务进程。

如果设置单独进程，那么相比直接在业务进程使用webview的场景，就需要多一个“与业务进程的IPC通信”。

 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/73.png" >



