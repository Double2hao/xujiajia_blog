#Raft协议详解
>  
 本文参考： 《大数据日知录》  


# 简介

Raft是一个一致性算法，它用于在分布式场景中保证多个副本之间的一致性。

>  
 对于”分布式场景“和”一致性“不是很理解的读者可以先看下笔者前一篇文章： 


在Raft中，实现多个副本的一致性的方式是，让多个副本之间的log数据达成一致。

# 基本概念

集群中的服务器有三个状态：Leader、Follower和Candidate。 每个状态的服务器会有不同的职责：

**Leader**：负责更新log数据，会先更新自己的本地的数据，然后同步到每个副本。 **Follower**：负责接收Leader的更新请求，更新自己本地的log数据。 **Candidate**：如果Follower一段时间没有收到Leader的心跳，那么就会认为系统中没有Leader，于是它就会把自己的状态变成Candidate，开始Leader的选举过程，直到选举结束，会一直处于Candidate这个状态。

### Term序列

Raft将整个系统执行时间划分成多个时间片段，每个时间片段对应一个Term编号，因此整个系统执行时间就是一个Term序列。 每次选举只会对应一个Term编号，如果选举失败，Term会递增，进行下一次选举。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039129560.png" alt="在这里插入图片描述">

# Leader选举

### 选举触发

Raft采用心跳机制来触发Leader的选举过程。Leader通过周期性地向其他服务器发送心跳来保持其Leader地位。 如果Follower过了一定时间段没有接收到任何心跳信息，则会认为Leader已经不存在（Leader可能是宕机，也可能是网络中断），然后就会启动新的Leader的选举过程。

### 选举策略
1. 每个服务器只能投一票。1. 每次选举，只有一台服务器获得到了大多数的投票，才能变成Leader。 如果集群中一共有5台服务器，那么至少有3台服务器选它，它才能成为leader。
### 选举过程

开始选举前，Follower增加Term编号进入Candidate状态，然后向集群内所有其他服务器发送让它成为Leader的请求。随后它就会一直处于Candidate状态，直到以下情况之一发生：
1. 它赢得了本次选举，成为了Leader。 Candidate接收到大多数服务器的投票就能成为Leader，在成为Leader之后，就会开始给其他服务器发送心跳来保持其Leader地位。1. 其他服务器赢得了选举，成为了Leader，并向它更新log信息。 如果其他服务器收到了大多数的投票成为了Leader，就会给此服务器发送心跳，此服务器确认成功后就会把自己的状态改回Follower。1. 经过一段时间后，没有Leader产生。 这种情况下，没有服务器成为Leader，一般是因为同事有多个Follower转化为Candidate状态，导致选票分流，没有一个服务器得到大多数的选票。这种情况下Candidate都会超时并增加自身Term编号重新进入新一轮的选举过程。 为了能尽可能减少这种情况发生的概率，Raft采取每个服务器超时时间都设定为随机值来解决。
# log同步

log同步笔者个人认为可以分为log复制和log提交两个部分：
- **log复制**：Leader将自己的Log复制到其他服务器的log数据中。- **log提交**：不管是Leader还是其他服务器，在log数据中写入了log并不代表log就一定会被执行了，只有log被提交了才会被执行。
## log复制

Leader会为每个Follower维护一个nextIndex，表示Leader给各个Follower发送的下一条log entry在log中的index，初始化为leader的最后一条log的下一个位置。 Leader给Follower发送复制Log的请求，带着(term_id, (nextIndex-1))， term_id即(nextIndex-1)这个槽位的log 的term_id，Follower接收到复制Log请求后，会从自己的Log中找是不是存在这样的Log entry，如果不存在，就给Leader回复拒绝消息，然后leader则将nextIndex减1，再重复，知道复制Log请求被接收。

以下图Leader和b为例： 初始化，nextIndex为11，leader给b发送复制Log请求(6,10)，b在自己log的10号槽位中没有找到term_id为6的log 。则给Leader回应一个拒绝消息。接着，Leader将nextIndex减1，变成10，然后给b发送复制Log请求(6, 9)，b在自己log的9号槽位中同样没有找到term_id为6的log 。循环下去，直到leader发送了复制Log请求(4,4)，b在自己log的槽位4中找到了term_id为4的log 。接收了消息。随后，leader就可以从槽位5开始给b推送日志了。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039130691.png" alt="在这里插入图片描述">

## log提交

假设现在有一个5台服务器的集群，目前每个服务器的log结构如下图。 在这个集群中，在index=7之前的log都是可以提交的，因为在5台服务器中，已经有3台已经正确复制了这些log。 而index=8的log是不可以提交的，因为并没有达到”大部分服务器复制了该log“的条件。

所以，在这种情况下，只有index=7 以及之前的log会被执行，index=8的log虽然已经复制在了部分服务器的log数据中，但是由于没有被提交，所以是不会执行的。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039131572.png" alt="在这里插入图片描述">

# 安全性

假设目前有一个情况，LeaderA在同步状态的时候突然处于失效状态，在它失效的这段时间，LeaderB被选举了出来，并且同步了部分信息。这时候LeaderA恢复了状态又被选为了新的Leader，此时，LeaderA会用自身的旧的Log覆盖LeaderB的新的Log，此时就无法达到安全性的保证。 为了达到真正的安全性，Raft增加了如下两个约束条件：
1. 限制了哪些服务器可以被选举成为Leader。 要求只有其Log包含了所有已经提交的Log的那些服务器才有权被选举为新的Leader。1. 限制了哪些Log可以被提交。 对于Leader来说，只有它自己已经提交过当前term的操作才能被提交。
# 集群变化同步策略

集群变化是指整个集群中服务器的增加或者减少。 在Raft中，集群变化的同步是用一个两阶段协议来实现的，假设老的集群信息为Cold，新的集群信息为Cnew，集群同步的过程如下：
1. Cold U Cnew 指两者同时存在 。1. Leader先将复制Cold U Cnew的请求发送给其他服务器，确定有一半服务器更新之后，提交更新Cold U Cnew的这个log。（这时候整个集群还是以Cold为准的）1. 然后再将复制Cnew的请求发送给其他服务器，在确定有一半服务器更新之后，就提交更新Cnew的这个log。1. 此时，由于整个集群已经以Cnew为标准了，所以即使是没有复制Cnew的log的服务器，也会以Cnew为标准。（由于之前已经同步过了Cold U Cnew，所以没有复制Cnew的log的服务器也是知道新的集群的配置的）
不使用二阶段协议在下面的场景下会有问题：
1. 假设此时新加入集群了10台机器。1. 在Leader发送复制Cnew的请求到服务器，已经有一半以上的服务器复制成功，因此提交了Cnew的log。1. 此时Leader宕机，那么对于没有复制到Cnew的log的服务器就不会向新加入集群的10台机器发送选举的请求，这样就导致了安全性问题。