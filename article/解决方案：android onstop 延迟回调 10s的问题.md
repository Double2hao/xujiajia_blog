#解决方案：android onstop 延迟回调 10s的问题
# 概述

近期碰到activity onstop延迟回调10s的问题。 关于问题的具体复现和原因，有博主总结的很不错，此处给上原链接： 

此文主要记录一下笔者目前认为的解决方案，以及各方案的优劣。

# 1、避免过度频繁或耗时的主线程操作

假设现在是ActivityOne跳转到ActivityTwo，那么有两种情况都会导致延迟执行：
1. ActivityOne频繁执行主线程操作，比如一些循环执行的动画操作。1. ActivityTwo在onCreate或者onStart中执行一些耗时或者频繁的主线程操作。
### 优势

从根源上解决了问题。

### 劣势
1. 去动原来的逻辑就意味着风险，改动越大风险越大。1. 实际项目中，模块间往往是解耦的。有可能ActivityOne是我负责的，而ActivityTwo则是其他团队负责了。1. 有些频繁或者耗时的主线程操作由于业务原因没法去掉，或者去掉的风险太高。
# 2、使用EventBus

假设现在是ActivityOne跳转到ActivityTwo，那么在ActivityTwo执行到onCreate的时候就抛出事件，然后在ActivityOne中接收事件，做与onstop中一样的处理。

# 优势

几乎不会影响代码原来的逻辑，风险很低。

# 劣势
1. 项目需要支持EventBus。（EventBus一旦滥用，代码会非常恶心，因此很多项目直接不允许用EventBus）1. 这种情况更适合一些公用的页面，比如登录页面，webview页面，视频播放页面等。在这些场景下，抛出这个事件才会比较合理。1. 如果原来没有这块逻辑，仍然需要在两个activity中都改。
# 3、监控所有activity的生命周期

假设现在是ActivityOne跳转到ActivityTwo，在ActivityOne中监控所有activity的生命周期，在ActivityTwo的onCreate执行到的时候，去执行onstop的逻辑。

# 优势

只需要在ActivityOne中新增逻辑即可，不需要动ActivityTwo。

# 劣势
1. 需要靠谱的识别activity的方式。如果仅使用ActivityTwo的类名作为识别方式，那么如果ActivityTwo一旦改名，这块逻辑就会有问题。1. 难以判断行为轨迹，假设现在有A,B,C三个activity。 我只想监控A到B的情况，如果我只判断B的onCreate的话，那么如果A到C再到B也会被监控到。