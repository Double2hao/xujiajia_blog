#ios lib运行找不到.h文件 “.h file not found”
# 问题

在运行lib target项目的时候有一些.h文件会找不到。但是在正常taget中是没有问题的。

# 解决方案

在Stack Overflow上查了解决方案是在lib的target的Build Setting中搜索“Header Search Path”配置。于是笔者配置了如下：(xxx表示库的名字) <img src="https://img-blog.csdnimg.cn/20190622102438867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> 但是在配置之后还是会有问题，查找了半天才找到问题所在。

我们可以对照下再正常项目的target中“Header Search Path”的配置如下： <img src="https://img-blog.csdnimg.cn/20190622102339480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

两者虽然在“Header Search Path”配置的内容是一样。 但是真正灰色字体显示的路径确实不一样的，在真正的项目中的开头是"/Users/xxx"，可以找到这个库的绝对路径。而在lib的target中，开头却是“/Headers/Public”。 说明“${PODS_ROOT}”在lib项目中是不生效的。 于是最终把地址改成了如下就可以了： <img src="https://img-blog.csdnimg.cn/2019062210294131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94dWppYWppYS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">