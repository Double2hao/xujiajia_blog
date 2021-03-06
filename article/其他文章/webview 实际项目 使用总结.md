#webview 实际项目 使用总结
# webview概述

webview在客户端使用非常广泛。 本文主要针对个人对webview的应用和学习对相关知识点做一个总结，如有整理的不对或者不同意见欢迎评论交流。

目前市场上的app使用场景有以下：（只是举例，并不代表所有场景）
1. 广告页面1. 活动页面1. 与客户端app功能关系不大，但是两端都要有的页面。例如：客服页面，申诉页面，签到页面等。
# webview优劣

使用webview主要有以下优势：
1. 不用跟客户端版本，开发灵活。随时可以发布新内容或者修改bug。1. 双端统一，一份代码在android与ios两端都可以使用。1. 业务上解耦，如广告投放的场景，只需要投放方给广告链接就可以，客户端不用关心其他。
劣势有以下几点：
1. 性能与使用流畅度肯定不如客户端。1. 涉及到客户端功能的支持，还是需要依赖客户端的发版。如增加对定位、传感器之类的支持。1. 不如客户端安全。需要限制跳转到外部页面，或者限制外部页面对客户端功能的调用。
# webview缓存策略

webview本身的性能是不如客户端的，使用缓存策略能一定程度上优化webview的使用体验。 缓存策略可以加快webview的加载网页的速度，减少白屏时间。 一般缓存策略有以下两种：
1. 缓存页面中的图片、视频等占用较大而不影响逻辑的资源。 在第二次加载同一个页面的时候可以更快。
>  
 页面中占用资源较大的一般都是图片视频，客户端缓存了之后，在下次访问的时候直接从本地获取，这样能够节省下网络下载的时间。 

1. 提前缓存整个页面中的资源，包括html，css，js和图片等。 这样在第一次加载的时候就可以直接从本地获取资源，节省了网络请求的时间。
>  
 直接缓存整个页面的资源有比较明显的劣势——页面会有更新不及时的问题。 因此，如果采用这个策略，一般需要满足以下几个条件： 
 - 页面本身必须是不需要及时更新，对错误有一定容忍度的场景，比如活动页面。如果是登录验证等对错误低容忍的页面则不适合。- 另外是需要自己控制页面资源的更新时间，比如可以在页面启动的时候更新资源。 


# webview黑白名单

黑白名单用来控制不同页面的不同权限，主要是为了安全。 更多情况下用在广告场景，因为如果都是内部自己开发的页面一般不需要从webview层面来限制权限。 webview中一般有以下场景：
1. 页面跳转的权限
>  
 假设现在对外投放一个广告。 这个广告在上线后更改了自己的逻辑，一进入页面就会跳转到微信或者他们自己的app，这样就等于是帮其他应用导流了。 很多时候这样的场景是不允许的，因此就需要白名单来控制该页面只允许在限定的url中跳转。 

1. 调用jsbridge的权限
>  
 jsbridge一般都是用来把客户端的部分能力提供给H5。 其中的能力很多都是可能涉及用户隐私或者app较重要的功能调用。 举个例子，比如在一些内部的活动页面可能通过jsbridge来获取用户的设备id，用户的定位等信息。但是毫无限制的把这样的功能开放给外部的广告页面，在安全上就值得商榷了。 


# webview中UA的使用场景

在webview中的页面可以通过userAgent来判断当前的浏览器版本，操作平台等信息。 比如可以判断当前H5是在自己的app中的webview打开的呢，还是在微信QQ的webview中打开的。 还是通过举例来说下UA会在什么场景下使用：
1. 判断版本
>  
 客户端不同版本的jsbridge之间可能会有调用差别，或者说有些接口被替换或者废弃了。 在这种情况下，在H5中可以通过UA中版本的判断来对不同的版本做兼容。 虽然这样的实现会给后期带入较大的维护成本，但是往往还是可以被当做一种兜底策略。 

1. 判断平台
>  
 假设现在有个H5页面，同时会投放到两个场景： 
 - 作为内部活动页。在这种场景中，能够通过jsbridge直接跳转到对应的客户端页面。- 作为外部的分享页面。在这种场景中，由于不存在需要jsbridge，因此逻辑就变成”跳转到对应的app或者提示下载“。 


# webview中jsBridge实现方式

### native调用js

直接使用客户端提供的接口调用js方法或者运行js代码即可。 （此接口中一般都有回调，不需要自己实现）

>  
 此处以Android为例，该接口为“evaluateJavascript(script: String, resultCallback: ValueCallback&lt;String!&gt;?)” 


### js调用native
1. 自定义url，通过scheme来区分jsbridge与一般url。
>  
 如 jsbridge://method?a=2&amp;b=3#h5MethodTag 

1. 使用客户端提供的接口在H5中注入native逻辑的调用方式。
>  
 android与ios的接口不一样，都需要单独适配。 此处以android为例，该接口为 “addJavascriptInterface(object: Any, name: String)” 


### js调用native的回调

native调用js的接口中本来就实现了回调，而js调用native的回调则需要开发者按照自己的策略实现。 此回调一般有两种实现方式：
1. native开发者手动调用实现。
>  
 假设，现在H5期望为客户端的gps增加监听器，要求每1秒能触发一次回调。 按照这个策略，就同时需要实现两个协议，一个用来让H5控制监听器的开启与结束，另一个用来定义监听器返回数据后的回调。假设两个协议的key分别为“getLocation”和“onGetLocation”。 整体流程如下： 
 - 由H5通过“getLocation”来触发该监听的打开- 客户端每1s触发一次js的“onGetLocation”来将数据返回给js。- 使用结束后，H5通过“getLocation”来关闭该监听器 

1. 在框架层面直接实现回调。
框架层实现回调后，可以让js直接调用一个接口来实现整个需求。大概代码如下：

```
nativeJsBridge.notify{<!-- -->'getLocation',new function(latitude，longitude){<!-- -->
//此处实现回调内容
}}

```

此回调实现的关键点其实是如何将js实现的这个回调传递给native。 其实只需要换个思路，不需要直接传回调，在js中保留该回调与一个long值的映射关系，然后将这个long值传给native，native触发回调的时候通过这个long值去触发即可。 大概流程如下：
1. js触发 “getLocation” 并传一个代表了该回调的long值给native。1. native在触发回调的时候，传给js 这个long值，和该回调中需要的参数（在此例子中是经度和纬度）。1. js在收到long值时，在本地的映射关系中找到这个function，并且将客户端传来的参数放入这个方法调用。这样就成功触发了回调。
# Cookie 同步问题

android和ios两端都需要注意一个cookie同步的问题，还是举例来说明下：

>  
 假设现在进入界面A，在A界面触发了登录，登录成功后设置了全局的cookie。 此时A界面的cookie还是未登录状态的。需要手动触发界面刷新，webview中的界面才能同步到全局的cookie。 


### android多进程cookie同步问题

假设现在有AB两个进程，启动B进程之前有cookie “user=1”，启动B进程之后A进程将全局cookie设置为“user=2”，此时B进程的cookie不会被更新，将会一直是“user=1”。 只有当B进程重启才能更新该cookie。

>  
 此处推荐文章： 


### ios wkwebview的cookie同步问题

ios中uiwebview的cookie是及时同步的，但是现在wkwebview性能更佳，官方更推荐使用wkwebview。 wkwebview中的cookie不是及时同步，还是举个例子。

假设目前全局的cookie为cookie “user=1”，再将全局的cookie设置成 “user=2”的时候，马上再打开wkwebview，此时wkwebview的cookie有可能还是“user=1”。

>  
 此处推荐文章： 


# 架构问题

### 各项目间的jsBridge如何解耦？

需要在设计框架的时候，框架的只实现部分基础功能的jsBridge，比如调起app，调起相册，获取位置信息等。

其他的诸如分享、登录等与其他业务耦合的功能在该应用实现的时候通过注入的方式实现。 与其他业务耦合度较高的功能，由业务方自己去实现自己的功能来注入。

### 如何避免业务之间jsBridge重复或者互相覆盖？

可以给每个业务设置一个namespace，每个业务的namespace不会重复，每个jsBridge必须在namespace内，或者直接让namespace作为jsBridge的key的前缀。