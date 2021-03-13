#多线程读写优化（双buff内存交换代替有锁设计）
# 例子（场景）

目前有线程ThreadA和ThreadB，一个队列Queue。ThreadA会对Queue进行入队操作，而ThreadB会对Queue进行出队操作。如下图：  <img src="https://img-blog.csdn.net/20180722093249239?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="这里写图片描述" title="">  一般情况下，我们都会直接给Queue上锁，这样就能保证多线程同时对Queue进行操作时不会有问题。  直接加上锁可以很容易就解决这个问题，但是也会带来其他的问题：入队操作一般几乎不耗时，而出队操作往往带有其他一系列逻辑操作，所以会比较耗时。因此ThreadA本来做完一系列入队操作可能只要3ms，但是由于等待ThreadB的锁的释放，可能多等待了200ms。如下图：  <img src="https://img-blog.csdn.net/20180722093824866?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="这里写图片描述" title="">

在这种情况下，如果ThreadA在入队操作还有其他逻辑，那么后面的逻辑会被延后200ms执行，这是完全没有必要的，因此便可以通过以下方式优化。

# 优化

直接使用两个Queue对象，一个只给ThreadA用来入队，一个只给ThreadB用来出队，这样入队和出队操作就可以分离，不用去争抢锁。  达到一定触发条件的时候两个Queue的内存就进行交换，原来入队的Queue变为出队的Queue，出队的Queue变成入队的Queue。这个触发条件可以由ThreadA来控制，在ThreadA认为不需要继续入队并且ThreadB的队列为空的时候，两个Queue可以进行交换。如下图：  <img src="https://img-blog.csdn.net/20180722094845395?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="这里写图片描述" title="">  这样之后，在时间上的表现就变为下图。对于ThreadA来讲，一次将几乎不耗时的入队操作做完，后面如果有其他逻辑可以不会被耽误。而对ThreadB来讲，本来执行的操作可能就比较耗时，等待ThreadA的入队操作时间也非常短，所以影响不大。  <img src="https://img-blog.csdn.net/20180722095046492?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="这里写图片描述" title="">

# 补充

可能读者不太理解为什么出队操作会那么耗时。因为上面是假设的出队与后续逻辑操作连在一起的情况。  那么是否意味着出队之后直接释放锁，这种情况就不适用了呢？不是的。  入队操作需要占用一次锁和释放一次锁，出队操作同样是的。如果每出队一次就需要占用和释放一次锁，那么如果有100个就需要占用和释放锁一百次，这是在数量较多的情况下是非常消耗资源的了。  因此把锁给去掉在这种情况下也是有相当的优化价值。