#Plugin is too old, please update to a more recent version 解决办法 2016.1.2
Android studio在更新了2.0之后，新建项目，遇到了如下问题：

 

Error:(1, 0) Plugin is too old, please update to a more recent version, or set ANDROID_DAILY_OVERRIDE environment variable to "300a703cef4ee5ebce05b6fd5b3a49f4e03961e2" &lt;a href="fixGradleElements"&gt;Fix plugin version and sync project&lt;/a&gt;&lt;br&gt;&lt;a href="openFile:D:\android-studio-location\ViewPager_fragment2\app\build.gradle"&gt;Open File&lt;/a&gt;

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911071190.png ">

 

解决办法就是改变项目的**build.gradle**中的内容，**需要把gradle的版本改为原来可以运行的项目的版本（笔者此处的为1.3.0）**，如下图所示：

 

**这是产生错误的地方：**

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911072201.png ">

 

 

**改正之后的结果：**

 

**<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911072752.png ">**

 

改过之后项目就不再报错了，其中具体原因就由读者自己去深究了。