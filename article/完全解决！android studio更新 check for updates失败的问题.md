#完全解决！android studio更新 check for updates失败的问题
笔者近期更新android studio的时候遇到了如下问题：Connection failed (connect timed out). Please check network connection and try again.

 

如下图：

 

<img alt="" class="has" src="https://img-blog.csdn.net/20151102091215011?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

经百度后，明白要有以下操作：

修改$ANDROID_STUDIO_HOME/bin/ 下的 studio.exe.vmoptions（或者studio64.exe.vmoptions）

 

配置后追加如下：

 

-Djava.net.preferIPv4Stack=true

-Didea.updates.url=http://dl.google.com/android/studio/patches/updates.xml

-Didea.patches.url=http://dl.google.com/android/studio/patches/

 

如下图：

 

<img alt="" class="has" src="https://img-blog.csdn.net/20151102091452757?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

 

<img alt="" class="has" src="https://img-blog.csdn.net/20151102091510744?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

 

但是接下来就是让人纠结的问题了——配置好了之后，android studio还是与之前一样不能更新！

那就再百度吧！接着笔者发现是因为之前为了要让sdk更新而修改了系统的hosts文件，把hosts文件的内容改回来就可以更新了。那就再试一下吧：

 

<img alt="" class="has" src="https://img-blog.csdn.net/20151102091729290?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

<img alt="" class="has" src="https://img-blog.csdn.net/20151102091923237?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

 

 

 

接着再试一下就可以更新了：

 

<img alt="" class="has" src="https://img-blog.csdn.net/20151102091947523?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">
