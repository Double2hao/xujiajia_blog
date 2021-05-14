#apache kafka 简介
# 概述

Apache Kafka 是分布式发布-订阅消息系统。 该项目的目标是为处理实时数据提供一个统一、高吞吐、低延迟的平台。 kafka合适也是最常见的使用场景就是日志投递。即适合对可靠性、持久性、吞吐量要求高的场景。

# 图解

下图摘自wiki： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040172000.png" alt="在这里插入图片描述"> 一般使用中会接触到的主要是以下几个名词： **Producer：** 消息的投递者 **Consumer：** 消息的消费者 **Topic：** Producer和Consumer交互的中间层。Producer每次需要选定将消息投递到哪个Topic上，而Consumer则需要设置好需要订阅哪些或者哪个Topic。 **Partition：** 是Topic的物理分区，一般情况下可以动态调整。一个Topic可以有多个partition。一个partition也可以在多个Topic中。

# 处理流程

1、Producer投递消息到对应的Topic。 2、Consumer订阅了固定的Topic，发现该Topic收到投递消息，于是Consumer拿到该消息开始处理。 3、如果Producer投递的速度超过了Consumer处理的速度，消息也不会丢失，没有来得及处理的消息会存储在partition上。开发者可以选择增添consumer的个数加快处理速度，或者在确保Producer投递速度肯定会降下来的情况下，让consumer自行慢慢处理。