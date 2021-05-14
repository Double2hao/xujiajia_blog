#Android Studio查看SHA1和MD5（附带keystore创建）
Android Studio中不能用查看MD5与SHA1的可视化界面，但我们可以通过命令的方式查看MD5与SHA1。

 

在windows中可以有两种方法：

打开windows的cmd或者直接在android studio 的terminal查看，大抵的操作都差不多。

 

首先要进入到自己的keystore文件的目录下，笔者的目录为D:\android studio-key。没有的自己可以去创建一个，当然此处也是可以使用系统默认的keystone文件，但是最好能自己搞一个，之后还可以用来发布apk，具体操作如图。

提醒一下：密码最好不要忘记。**(倘若使用系统默认的keystore就可以不用这一步了，直接可以跳到下一步)**

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911652880.png ">

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911654041.png ">

 

 

打开terminal或者cmd进入到keystore目录下，然后在终端中输入keytool -v -list -keystore key.jks（也可以是keytool -v -list -keystore debug.keystore）命令即可查看调试环境下的MD5与SHA1。如图：

 

（如果要使用系统默认的keystone文件，直接到用户目录的.android目录下执行keytool -v -list -keystore debug.keystore，笔者的目录是C:\Users\佳佳\.android）

**系统默认的keystore的密码为android。**

 

**（后来经过测试，建议新手还是使用系统默认的keystone文件，倘若使用自己的，就会牵扯到debug和realease的问题，稍微有点麻烦）**

 

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911654902.png ">

 

<img alt="" class="has" src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911655533.png ">

 

 

提示：keytool 是jdk中的一个工具，如果提示'keytool' 不是内部或外部命令，也不是可运行的程序 或批处理文件。"则可以将你的jdk安装目录下的bin目录添加到系统变量中即可。

 

 