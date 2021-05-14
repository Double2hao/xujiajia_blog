#git rebase 和merge 区分
>  
 本文参考  


# rebase和merge区分

## 相同点

两者都可以合并代码。

## 不同点

比如现在在某个子分支执行git rebase(merge) master操作。

merge:将在子分支的所有提交记录成一次commit，保留在记录中。（下图的E即为该记录） rebase:不会保留commit记录，直接将分支中的内容排到master的记录之后 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210038802860.png" alt="在这里插入图片描述"> merge： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210038809691.png" alt="在这里插入图片描述"> rebase： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210038811402.png" alt="在这里插入图片描述">

# 使用场景：

### 场景一

>  
 一直在某个子分支开发功能。你还没开发完，然后你的同事告诉你，你引用的其他模块的内容有更新，你需要拉取一下最新的master代码更新一下该的模块。 


推荐使用rebase。该模块的内容更新和你功能无关，合并代码也不会影响你的功能，无需保留该记录。如果只使用merge，在开发周期长的情况下，会创造很多无用的commit记录。

### 场景二

>  
 你和同事两个人在开发同一个模块，你同事开发的部分功能已经合并到了master，通知你更新一下最新的模块代码。 


推荐使用merge。你和同事开发的是同一模块，很可能他的某个改动会导致你的功能出问题，如果出了问题，保留记录能便于后期排查问题。（如果从逻辑上可以判断不会有影响，那么使用rebase也可以）

### 场景三

>  
 现在有个子分支模块已经开发完，master分支需要将这个分支的内容合并进来。 


推荐使用merge。使用merge可以留有提交记录，如果该模块出了问题，方便后面排查问题。