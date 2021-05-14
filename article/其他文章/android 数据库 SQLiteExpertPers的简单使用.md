#android 数据库 SQLiteExpertPers的简单使用
其实SQLiteExpertPers的使用也是比较简单，此文主要只是给新手一个捷径。

SQLite Expert Professional是一款可视化的数据库管理工具，允许用户在 SQLite 服务器上执行创建、编辑、复制、提取等操作。

SQLiteExpertPers的下载就不说了，直接百度一下有很多，本文主要如题说一下SQLiteExpertPers的简单使用。

 

1、需要使用DDMS从虚拟机（也可以是手机，笔者手机开不了权限）中导出db格式的数据库，android studio点击图中右上角的小机器人。

 

<img alt="" class="has" height="390" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910905290.png " width="700">

 

2、在使用DDMS找到你的app中的数据库文件，一般是在\data\data\com.example.xxxx 中后面的文件取决于你的app的包名，找到后导出。（如果出现的手机中的data文件无法打开的情况，那么便是你的权限的问题了，具体可能是比较麻烦的问题，可以百度，嫌麻烦的可以直接使用虚拟机。）

 

<img alt="" class="has" height="390" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910909551.png " width="700">

 

 

3、打开SQLiteExpertPers，并且使用它打开你刚刚导出的db后缀名的数据库文件。

 

<img alt="" class="has" height="390" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910911132.png " width="700">

 

 

 

最终就可以看到你的数据库的可视化界面啦。

 

<img alt="" class="has" height="390" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910912653.png " width="700">