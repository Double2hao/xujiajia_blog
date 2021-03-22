#Java8 JVM内存结构变了，永久代到元空间
>  
 原文地址：https://blog.csdn.net/wo541075754/article/details/102679905 


# JVM内存结构的细化

为了更细化的讲解，我们将该图进行进一步的优化调整。针对java7及以前版本的细化。

看出变化了吗？堆和方法区连在了一起，但这并不能说堆和方法区是一起的，它们在逻辑上依旧是分开的。但在物理上来说，它们又是连续的一块内存。也就是说，方法区和前面讲到的Eden和老年代是连续的。

在继续进行下去之前，我们先来理解两个概念：规范和实现。

# 规范和实现

针对Java虚拟机的实现有专门的《Java虚拟机规范》，在遵守规范的前提下，不同的厂商会对虚拟机进行不同的实现。 就好比开发的过程中定义了接口，具体的接口实现大家可以根据不同的业务需求进行实现。

PS：大家都有必要了解一下《Java虚拟机规范》，关注公众号“程序新视界”，回复“002”获得Java SE 7的虚拟机规范PDF版。

我们通常使用的Java SE都是由Sun JDK和OpenJDK所提供，这也是应用最广泛的版本。而该版本使用的VM就是HotSpot VM。通常情况下，我们所讲的java虚拟机指的就是HotSpot的版本。

# 永久代（PermGen）

上面理解了规范和实现之后，来看认识一个概念“永久代(Permanet Generation，也称PermGen)”。对于习惯了在HotSpot虚拟机上开发、部署的程序员来说，很多都愿意将方法区称作永久代。

本质上来讲两者并不等价，仅因为Hotspot将GC分代扩展至方法区，或者说使用永久代来实现方法区。在其他虚拟机上是没有永久代的概念的。也就是说方法区是规范，永久代是Hotspot针对该规范进行的实现。

理解上面的概念之后，我们对Java7及以前版本的堆和方法区的构造再进行一下变动。

再重复一遍就是对Java7及以前版本的Hotspot中方法区位于永久代中。同时，永久代和堆是相互隔离的，但它们使用的物理内存是连续的。

永久代的垃圾收集是和老年代捆绑在一起的，因此无论谁满了，都会触发永久代和老年代的垃圾收集。

但在Java7中永久代中存储的部分数据已经开始转移到Java Heap或Native Memory中了。比如，符号引用(Symbols)转移到了Native Memory；字符串常量池(interned strings)转移到了Java Heap；类的静态变量(class statics)转移到了Java Heap。

然后，在Java8中，时代变了，Hotspot取消了永久代。永久代真的成了永久的记忆。永久代的参数-XX:PermSize和-XX：MaxPermSize也随之失效。

# 元空间(Metaspace)

对于Java8，HotSpots取消了永久代，那么是不是就没有方法区了呢？当然不是，方法区只是一个规范，只不过它的实现变了。

在Java8中，元空间(Metaspace)登上舞台，方法区存在于元空间(Metaspace)。同时，元空间不再与堆连续，而且是存在于本地内存（Native memory）。

本地内存（Native memory），也称为C-Heap，是供JVM自身进程使用的。当Java Heap空间不足时会触发GC，但Native memory空间不够却不会触发GC。

针对Java8的调整，我们再次对内存结构图进行调整。

元空间存在于本地内存，意味着只要本地内存足够，它不会出现像永久代中“java.lang.OutOfMemoryError: PermGen space”这种错误。看上图中的方法区，是不是“膨胀”了。

默认情况下元空间是可以无限使用本地内存的，但为了不让它如此膨胀，JVM同样提供了参数来限制它使用的使用。

-XX:MetaspaceSize，class metadata的初始空间配额，以bytes为单位，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当的降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize（如果设置了的话），适当的提高该值。 -XX：MaxMetaspaceSize，可以为class metadata分配的最大空间。默认是没有限制的。 -XX：MinMetaspaceFreeRatio,在GC之后，最小的Metaspace剩余空间容量的百分比，减少为class metadata分配空间导致的垃圾收集。 -XX:MaxMetaspaceFreeRatio,在GC之后，最大的Metaspace剩余空间容量的百分比，减少为class metadata释放空间导致的垃圾收集。

# 永久代为什么被替换了

思考一下，为什么使用元空间替换永久代？

表面上看是为了避免OOM异常。因为通常使用PermSize和MaxPermSize设置永久代的大小就决定了永久代的上限，但是不是总能知道应该设置为多大合适, 如果使用默认值很容易遇到OOM错误。

当使用元空间时，可以加载多少类的元数据就不再由MaxPermSize控制, 而由系统的实际可用空间来控制。

更深层的原因还是要合并HotSpot和JRockit的代码，JRockit从来没有所谓的永久代，也不需要开发运维人员设置永久代的大小，但是运行良好。同时也不用担心运行性能问题了,在覆盖到的测试中, 程序启动和运行速度降低不超过1%，但是这点性能损失换来了更大的安全保障。