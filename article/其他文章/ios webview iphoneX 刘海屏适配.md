#ios webview iphoneX 刘海屏适配
>  
 参考文章： 


# 问题

webview 在iphoneX中的一些表现会不满足业务预期，如下广告，头部和底部都会留有空白，而业务方真正期望是填满整个屏幕。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911200610.png " alt="在这里插入图片描述"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911202111.png " alt="在这里插入图片描述"> 期望效果如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911203622.png " alt="在这里插入图片描述"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911207223.png " alt="在这里插入图片描述">

# 解决方式

## H5解决（优先使用）

在需要适配iphone X的头部添加一行代码，这也是苹果官方提供的适配iphoneX的方式。

```
&lt;meta name="viewport" content="viewport-fit=cover" /&gt;

```

## native解决（H5有输入框情况不推荐）

对于某些情况，比如开机屏广告页面。这些页面的H5是第三方的，不可能要求每个第三方页面都自己去适配iphoneX。因此，只能通过native的方式来解决。 需要使用wkwebview并且设置如下代码：

```
self.wkwebview.scrollView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;

```

>  
 使用wkwebview native的方式去解决会引入其他问题。 这行代码是禁止wkwebview中的uiScrollView自动适配大小，如果页面中有输入框，在键盘把uiScrollView顶上去的情况下，wkwebview中的H5不会自动弹回。 所以还是建议能使用H5解决就让H5解决。 
