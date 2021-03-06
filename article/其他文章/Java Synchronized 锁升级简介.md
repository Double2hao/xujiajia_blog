#Java Synchronized 锁升级简介
# 锁升级顺序
1. 偏向锁1. 轻量级锁1. 自旋锁1. 重量级锁
# 偏向锁

如果一个线程获得了锁，再次请求的时候就不需要再去获取锁。如果发现有其他线程来获取这个锁，就升级为轻量级锁。

### 理论基础：

在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁的代价而引入偏向锁。

# 轻量级锁

通过使用CAS来获取锁，如果CAS失败就升级为自旋锁。

>  
 相关链接： 


### 理论基础

对绝大部分的锁，在整个同步周期内都不存在竞争

# 自旋锁

线程去获取锁的时候，默认认为锁的使用方会很快释放锁，所以就一直循环获取这个锁（自旋），一般不会获取太久，可能是50个或者100个循环。如果自旋获取锁失败，就自动升级为重量级锁。

### 理论基础

在大多数情况下，线程持有锁的时间都不会太长，线程的挂起和恢复本身也是一种损耗，在这种情况下频繁挂机和恢复就显得不值得，因此就用自旋的方式来避免频繁地线程调度。

# 重量级锁

线程获取锁的时候如果发现已经被占用，那么这个线程就挂起，一直到锁的使用方使用结束之后才会唤醒这个线程。