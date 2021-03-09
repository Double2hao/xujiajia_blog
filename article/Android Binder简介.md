#Android Binder简介
>  
 参考文章：   


# 基本概念

Binder：是Android进程间通信的一种机制，主要包括四个组件，用户空间的三个组件Client、Server、Service Manager和内核空间的一个组件Binder驱动。

Client：进程间通信的客户端 Server：进程间通信的服务端 Service Manager：是一个守护进程，用来管理Server，并向Client提供查询Server接口的能力 <img src="http://hi.csdn.net/attachment/201107/19/0_13110996490rZN.gif" alt="在这里插入图片描述">

# Binder驱动

Binder驱动是Binder通信的核心，尽管名叫‘驱动’，实际上和硬件设备没有任何关系，只是实现方式和设备驱动程序是一样的它工作于内核态，提供open()，mmap()，poll()，ioctl()等标准文件操作。 个人认为，binder驱动和管道、Socket、内存共享一样都是一种进程间通信方式，只是binder驱动是Android Binder机制专用的通信方式。 效率对比如下：

|IPC|数据拷贝次数
|------
|共享内存|0
|binder驱动|1
|Socket/管道/消息队列|2

那么binder驱动是怎么做到的呢？ 简单来说就是，发送方和内核空间的内存映射到了同一块物理地址，接收方到内核中去拿数据的时候，实际是从发送方的用户空间拷贝到接收方的用户空间，这样就少了一层用户空间到内核空间的拷贝。

>  
 这里也提一下管道的拷贝，首先是发送方从用户空间拷贝到内核空间，然后接收方从内核空间拷贝到用户空间，因此这就有两次拷贝。 对linux进程不了解的可以看一下笔者之前的文章： 


# 一般流程

Client、Server和Service Manager属于三个不同的进程，对于开发者来说，最终需要的是使用binder实现Client和Server两个进程之间的通信。 而对于其实际情况来讲，其实会有两个binder驱动对象: 一、Client和Server之间的Binder驱动对象，这个对象存储在Service Manager中，Client和Server通过与Service Manager通信可以注册和获取这个对象。 二、Service Manager与其他用户进程间的Binder驱动对象，用于实现其他用户进程和Service Manager之间的进程间通信。Server和Service Manager通信可以注册它需要的binder驱动对象，Client与Service Manager通信可以获取它需要的binder驱动对象。

一般流程如下： 1、首先系统为Service Manager创建一binder驱动对象，用于实现其他用户进程和Service Manager之间的进程间通信。

2、Server创建Binder实体，为其取一个字符形式，可读易记的名字，将这个Binder连同名字以数据包的形式通过Binder驱动发送给SMgr，通知SMgr注册一个名叫张三的Binder，它位于某个Server中。驱动为这个穿过进程边界的Binder创建位于内核中的实体节点以及SMgr对实体的引用，将名字及新建的引用打包传递给SMgr。SMgr收数据包后，从中取出名字和引用填入一张查找表中。

3、Server向SMgr注册了Binder实体及其名字后，Client就可以通过名字获得该Binder的引用了。Client也利用保留的0号引用向SMgr请求访问某个Binder：我申请获得名字叫张三的Binder的引用。SMgr收到这个连接请求，从请求数据包里获得Binder的名字，在查找表里找到该名字对应的条目，从条目中取出Binder的引用，将该引用作为回复发送给发起请求的Client。
