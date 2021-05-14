#iOS 10 is the maximum deployment target for 32-bit targets
# 问题

近期创建一个新项目引入了一些库，使用脚本打包lib的时候，发生了类似如下的报错：

>  
 iOS 10 is the maximum deployment target for 32-bit targets normal armv7 objective-c com.apple.compilers.llvm.clang.1_0.compiler 


检查了半天，发现是ios SDK版本的问题，新项目默认版本是ios 12，而库对之前的版本也是有支持的。 直接在找到项目的target，在General目录下选择Deployment Target为最老的版本即可。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2150.png" alt="在这里插入图片描述">