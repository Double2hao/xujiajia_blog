#android Activity A和B互相跳转 生命周期的变化
# 前提概要

此题出自笔者网上看到的一道面试题，原题如下：

```
Activity A 跳转到 Activity B，生命周期的执行过程是啥？


```

虽然笔者专门花时间了解过Activity生命周期以及启动模式等等，但是一下子问我这个跳转的生命周期还真有点不确定，于是，笔者就做了一个demo，彻底了解了一下Activity之间互相跳转的时候生命周期的变化。

# 打开APP，进入ActivityA

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDI1MTUxOTU2NjAy" alt="这里写图片描述">

## 生命周期如下：

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDI1MTUyMDA3OTE2" alt="这里写图片描述">

# ActivityA跳转到ActivityB

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDI1MTUyMDE1MTgx" alt="这里写图片描述">

## 生命周期如下：

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDI1MTUyMDIzMTk3" alt="这里写图片描述">

# ActivityB按back键返回

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDI1MTUyMDMzODM4" alt="这里写图片描述">

## 生命周期如下：

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDI1MTUyMDQyNjAz" alt="这里写图片描述">

# ActivityA按back键返回

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDI1MTUyMDUwMjQ0" alt="这里写图片描述">

## 生命周期如下：

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYxMDI1MTUyMDU5MzY5" alt="这里写图片描述">
