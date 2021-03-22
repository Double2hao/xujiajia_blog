#完全理解android Activity启动模式LauchMode （深入Activity与任务栈）
之前笔者已经讲过了LauchMode的作用，以及尽量避开栈的概念使用GIF图片的方式尽可能简单地阐述了一下Activity的启动模式，这篇文章就再次深入，好好讲一下在各种启动模式下，Activity与任务栈到底是如何作用的。

如果还是刚入门的读者，建议还是先看一下笔者的前一篇文章。

上一篇文章地址：

 

**任务栈：**（笔者此处就复制一下官方文档中的解释）

如果对任务栈有深入了解兴趣的，可以看一下官方译文：



 

当前 Activity 启动另一个 Activity 时，该新 Activity 会被推送到堆栈顶部，成为焦点所在。 前一个 Activity 仍保留在堆栈中，但是处于停止状态。Activity 停止时，系统会保持其用户界面的当前状态。 用户按“返回”按钮时，当前 Activity 会从堆栈顶部弹出（Activity 被销毁），而前一个 Activity 恢复执行（恢复其 UI 的前一状态）。 堆栈中的 Activity 永远不会重新排列，仅推入和弹出堆栈：由当前 Activity 启动时推入堆栈；用户使用“返回”按钮退出时弹出堆栈。 因此，返回栈以“后进先出”对象结构运行。 图 1 通过时间线显示 Activity 之间的进度以及每个时间点的当前返回栈，直观呈现了这种行为。

<img alt="" class="has" src="https://img-blog.csdn.net/20160605154735830?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center">

**图 1.** 显示任务中的每个新 Activity 如何向返回栈添加项目。 用户按“返回”按钮时，当前 Activity 随即被销毁，而前一个 Activity 恢复执行。如果用户继续按“返回”，堆栈中的相应 Activity 就会弹出，以显示前一个 Activity，直到用户返回主屏幕为止（或者，返回任务开始时正在运行的任意 Activity）。 当所有 Activity 均从堆栈中删除后，任务即不复存在。

 

**四种启动模式：**

 

**standard:标准模式，这也是系统的默认模式。**

在这种模式下，谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈中。

**例子：**

1、Activity A在任务栈S1中，Activity B在任务栈S2中，那么如果是A 启动了标准模式的C，那么C就会在S1栈中，如果是B启动了C就会在S2栈中。

**singleTop：栈顶复用模式。**

这种模式算是standard的优化版。仍然是谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈中。不同点就是多了一个判断。

**例子：**

1、目前S1栈内的情况是ABCD，此时再启动Activity D，如果启动模式是standard，那么栈内情况就是ABCDD。但是如果是singleTop模式，栈内情况就仍然还是ABCD。

注意：此处并不是把原来的D摧毁之后重新创建了一个D，而是直接把之前创建过的D拿过来用。

**singleTask：栈内复用模式。**

这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity就不会重新创建实例，而是把之前创建过的实例拿过来用。但是如果不存在该Activity的实例，就会重新开启一个栈，并把该Activity放入。

singleTask还默认具有clearTop的效果，会导致栈内所有在复用的Activity之上的Activity全部出栈。

**例子：**

1、当前S1任务栈中情况为ABC，此时启动站内复用模式的Activity D，由于本来的栈中并不存在D，那么此时就会新建一个任务栈S2，并且创建D的实例放入S2。

2、当前任务栈S1的情况为ADBC。

如果启用标准模式的Activity D，那么此时S1内的情况变为ADBCD。

而启动站内复用模式的Activity D，根据栈内复用的原则，D不会重新创建，系统会把D重新切换到栈顶。原则上此时栈内情况应该变为ABCD，但是由于singleTask默认具有clearTop的效果，所以最终结果应该是AD。

**singleInstance：单实例模式。**

望文生义，即一个任务栈中只会存在一个单一的实例。当然也不仅仅如此，由于栈中只会存在一个实例，所以由这个实例打开的的任何一个Activity都会在单独的任务栈中打开。

**例子：**

1、当前任务栈S1的情况是ABC**（ABC启动模式均为standard或者****singleTop****）**，此时启动单实例的Activity D，就会新建一个任务栈S2并把D放入。从Activity D再次打开ABC中任何一个，就会新建一个S3的任务栈，并且把这个Activity放入。此时就会存在三个任务栈S1、S2、S3。

2、当前任务栈S1的情况是ABC**（A的启动模式为singleTask）**，此时启动单实例的Activity D，就会新建一个任务栈S2并把D放入。此时从Activity D打开Activity A，会复用S1中的A。也就是说此时还是只有两个任务栈S1和S2。但是还是遵从singleinstance“由这个实例打开的的任何一个Activity都会在单独的任务栈中打开”的原则。