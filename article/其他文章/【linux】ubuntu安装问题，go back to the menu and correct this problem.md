#【linux】ubuntu安装问题，go back to the menu and correct this problem
如题，安装linux系统的时候遇到了如下问题：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039775150.png" alt=""> 

 

**翻译一下：**

go back to the menu and correct this problem <br style="color:rgb(51,51,51); font-family:'Helvetica Neue',Helvetica,Arial,sans-serif; font-size:14px; line-height:24px"> 返回菜单和纠正这个问题 <br style="color:rgb(51,51,51); font-family:'Helvetica Neue',Helvetica,Arial,sans-serif; font-size:14px; line-height:24px"> the partition table format in use on your disks normally requires you to create a separate partition for boot loader code. this partition should de marked for use as a "reserved bios boot area" and should de at least 1 mb in size. note that this is not same as a partition mounted in /boot. <br style="color:rgb(51,51,51); font-family:'Helvetica Neue',Helvetica,Arial,sans-serif; font-size:14px; line-height:24px"> 在使用你的磁盘通常要求您创建启动引导代码一个单独的分区的分区表的格式。这个分区应标记为使用作为一个“保留BIOS启动区”，应至少有1 MB的大小。请注意，这是不是安装在/boot分区相同。 <br style="color:rgb(51,51,51); font-family:'Helvetica Neue',Helvetica,Arial,sans-serif; font-size:14px; line-height:24px"> if you do not go back to the partitioning menu and correct,boot loader installation may fail later,although it may still be possible to install the loader to a partition. <br style="color:rgb(51,51,51); font-family:'Helvetica Neue',Helvetica,Arial,sans-serif; font-size:14px; line-height:24px"> 如果你不回到分区菜单和正确的引导装载程序，安装可能会失败后，尽管它可能仍然可以安装到一个分区的装载机。 

 

## 解决方法：

##  1.新建分区容量16或32Mb的空间 (它说是至少1Mb,为了以防万一就多分点呗，我分了32MB)。

##  2.新建分区位置-&gt;起始；主分区 

##  3.用于-&gt;biosgrup