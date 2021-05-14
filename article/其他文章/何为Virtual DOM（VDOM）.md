#何为Virtual DOM（VDOM）
# 前言

用vue已经比较久了，也是总是会接触到VDOM，但是一直有点不太理解为什么要这么设计，于是稍微学习记录一下。

>  
 参考文章： 


# 更新DOM的三种方式

1、重新加载，全局刷新 2、单个DOM节点有变化就更新 3、使用VDOM，更新界面中变化的节点

## 1、重新加载，全局刷新

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/730.png" alt="在这里插入图片描述"> 页面中的数据发生了变化，就重新请求页面，把整个DOM刷新一遍。 优点：简单。 缺点：对性能影响比较大。假设我界面中只有一个DOM节点有变化，却需要重新家在界面刷新整个DOM，显然性价比很低。

## 2、单个DOM节点有变化就更新

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/731.png" alt="在这里插入图片描述"> 开发者借助框架，监听数据的变更，在数据变更后更新对应的 DOM 节点。 优点：在更新的节点比较少的情况下性价比很高，不需要重新加载界面刷新整个DOM。 缺点：在更新的节点比较多的情况下性价比很低，更新多少个节点就要操作DOM多少次，还不如直接重新加载界面。

## 3、使用VDOM，更新界面中变化的节点

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/732.png" alt="在这里插入图片描述"> 初始渲染时，首先将数据渲染为 Virtual DOM，然后由 Virtual DOM 生成 DOM。 数据更新时，渲染得到新的 Virtual DOM，与上一次得到的 Virtual DOM 进行 diff，得到所有需要在 DOM 上进行的变更，然后在 patch 过程中应用到 DOM 上实现UI的同步更新。 优点：可以算是第二种方式的优化，在有多个节点需要刷新的情况下，由于通过查找VDOM的diff来一次更新，所以只需要操作一次DOM。 缺点：相比前两种方式，对开发者的要求更高，需要了解控制更新DOM的时机。在笔者vue的开发中经常碰到界面没有及时更新的情况。

# 典型场景对比

以下场景仅仅局限于一般情况，特殊场景可能并非如下所说，读者需要自行选择接受。

>  
 一、DOM中有10000个节点，有1个节点需要更新。 


1、直接重新加载界面，刷新DOM。 2、节点被修改后直接更新，操作DOM 1次。 3、通过VDOM对比diff更新DOM，操作DOM 1次。

性能: 2&gt;3&gt;1

>  
 二、DOM中有10000个节点，有1000个节点需要更新。 


1、直接重新加载界面，刷新DOM。 2、节点被修改后直接更新，操作DOM 1000次。 3、通过VDOM对比diff更新DOM，操作DOM 1次。

性能: VDOM性能最高，1和2性能都比较低。