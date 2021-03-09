#android webView 不能上网 无法打开网页的问题
近期学习WebView也是遇到了一个问题，花自己比较多的时间查询，最后发现仅仅是因为大小写的问题导致不能上网，也是心累，自己平时写代码当更加注意才是，在此分享一下遇到的问题，希望大家遇到了都能随即解决，接下来看代码。

 

主要是AndroidManifest.xml中对权限的设置。

<img alt="" class="has" src="https://img-blog.csdn.net/20150905110811327?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

<img alt="" class="has" height="480" src="https://img-blog.csdn.net/20150905110938517?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="270">

 

 

<img alt="" class="has" src="https://img-blog.csdn.net/20150905110837028?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

<img alt="" class="has" height="480" src="https://img-blog.csdn.net/20150905110944459?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center" width="270">

 

本人博客，android均为新手，闻过则喜，望前辈不吝指点。
