# 概述
一直想要备份下自己CSDN的文章，目前光原创的就有300多篇，完全手动备份太耗时间，于是准备写代码来实现。
大概思考了下，备份差不多需要以下几部分：
1. 备份文章的markdown文件
2. 备份所有文章中的图片（毕竟很多都是亲手画的，都是心血）
3. 替换所有markdown文件中图片的url

# 最终效果
最终备份了300多篇文章和400多张图片，总共700多个文件，如下图：

![在这里插入图片描述](https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/20210313192451555.jpg)

下载所有图片和替换所有url的总耗时 2分29秒，如下图：
![在这里插入图片描述](https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/20210313192507351.jpg)


# 备份markdown
这个部分直接在github上找到了现有的项目，相关项目还是挺多的，有兴趣的可以自行在github上找了项目试试。

笔者用的项目如下：
> [https://github.com/Tansty/CSDN-spider](https://github.com/Tansty/CSDN-spider)

# 备份图片和替换图片url
这部分更需要定制化，笔者就自己动手写了，笔者的这个项目只支持如下格式的图片引用：
```xml
<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwODEyMTQzNjAwNTk1" alt="这里写图片描述">
```

java项目地址：
> [https://github.com/Double2hao/TransformMarkDownImg.git](https://github.com/Double2hao/TransformMarkDownImg.git)

### 大致逻辑
1. 解析找到图片url
2. 下载图片（自定义图片名）
3. 替换markdown中的图片url
