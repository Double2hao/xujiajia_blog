#Android 热修复原理（java类加载机制）
>  
 原文地址：https://www.jianshu.com/p/cb1f0702d59f 


# 热修原理（java类加载机制）

DexClassLoader在加载一个类时会先从自身DexPathList对象中的Element数组中获取（Element[] dexElements）到对应的类，之后再加载。采用的是数组遍历的方式，不过注意，遍历出来的是一个个的dex文件。

在for循环中，首先遍历出来的是dex文件，然后再是从dex文件中获取class，所以，我们只要让修复好的class打包成一个dex文件，放于Element数组的第一个元素，这样就能保证获取到的class是最新修复好的class了（当然，有bug的class也是存在的，不过是放在了Element数组的最后一个元素中，所以没有机会被拿到而已）。

>  
 Tinker是将patch.dex与app中的classes.dex合并后的全量dex插在数组的前面。 
