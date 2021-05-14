#fresco要点记录
>  
 本文参考 https://github.com/facebook/fresco https://juejin.im/post/5a7568825188257a7a2d9ddb 


# 概述

近期使用到fresco，各要点总结记录下。本文内容多为转载或翻译。

# 官方描述特点

fresco官方的描述中主要有以下几个特点：
1. 原生的bitmap在被渲染出来后，native的资源就会被回收。这个操作是在UI线程进行的，因此会造成卡顿。 fresco中bitmap在被渲染出来后不自动回收native资源，由fresco框架来控制回收时机。1. Bitmap解码是非常消耗CPU资源的，当消耗过大时会引起UI阻塞。 在fresco中就对解码后的bitmap进行了缓存，避免解码造成UI卡顿。
# 关键类

**DraweeView**：继承于ImageView，只是简单的读取xml文件的一些属性值和做一些初始化的工作，图层管理交由Hierarchy负责，图层数据获取交由负责。 **DraweeHierarchy**：由多层Drawable组成，每层Drawable提供某种功能（例如：缩放、圆角）。 **DraweeController**：控制数据的获取与图片加载，向pipeline发出请求，并接收相应事件，并根据不同事件控制Hierarchy，从DraweeView接收用户的事件，然后执行取消网络请求、回收资源等操作。 **DraweeHolder**：统筹管理Hierarchy与DraweeHolder。 **ImagePipeline**：Fresco的核心模块，用来以各种方式（内存、磁盘、网络等）获取图像。 **Producer/Consumer**：Producer也有很多种，它用来完成网络数据获取，缓存数据获取、图片解码等多种工作，它产生的结果由Consumer进行消费。 **IO/Data**：这一层便是数据层了，负责实现内存缓存、磁盘缓存、网络缓存和其他IO相关的功能。

# 架构

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911389140.png " alt="在这里插入图片描述">