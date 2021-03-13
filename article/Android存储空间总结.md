#Android存储空间总结
# 概述

Android存储在6.0之前，真的可以用“混乱不堪”来形容。 造成的原因很多，个人认为有以下几点：
1. android早期手机本身的内存很小，大部分app的读写都在sd卡内操作。1. 用户权限意识薄弱，android 6.0之前直接在安装时申请权限，用户往往在未知情况下就给予了存储权限。1. app为了使自己的应用数据好看，直接将资源存储到sd卡，存储在sd卡的文件将不会计算成该app的内存占用。1. app处于卖方市场。如果不给存储权限用户就无法使用app部分甚至全部功能。
目前，随着Android越发完善，官方也有了自己推荐的存储方案。作为一个android开发者，非常有比较了解一下。

>  
 官方文档： 


笔者根据自己的日常使用经验以及官方文档做本文总结，如果有理解有误的地方欢迎评论交流。

# 存储方案优劣

### 内部存储

只有自己可以访问，其他应用不能访问。自己访问不需要权限。

>  
 android api一般是：context.getCacheDir() 和 context.getFilesDir() 


**使用场景**
- 小程序、小游戏 （持久文件，适合存储在context.getFilesDir()） 与app相关联，app卸载后就没有用了。- 缓存图片、视频 （临时文件，适合存储在context.getCacheDir()） 随时可以清理掉，清理掉后就是走冷启动逻辑。
**优势**
1. 不需要申请权限，用户无感知。1. 不会影响手机存储，app删除，即相关资源一起删除。
**劣势**
1. 无法永久化存储，app删除后会跟着删除。1. android手机存储不够时，应用的缓存文件会被删除，回收空间。
### 外部存储

获得读写权限后，大家都可以读写。

>  
 android api一般是：Environment.getExternalStorageDirectory() 


**使用场景**
- 照片、视频、文档等本地存储 需要放到外存，便于用户分享和使用。
**优势**
1. 除非用户主动删除，否则可以一直存储在用户手机中。 （这个用法很多，比如多个应用可以共享一份资源，不用每个应用都维护一份；当应用删除后第二次安装，能保留部分用户数据以及缓存）1. 获得权限后也意味着可以读取手机中的其他内容，如照片，视频等。
**劣势**
1. 需要申请用户权限，部分用户会选择拒绝。1. sd卡取出后，就无法访问。 这个问题主要在较老的机型上，目前大部分手机都没有sd卡了。1. 由于app被删除时外存中的数据不会自动被删除，因此就会成为用户的“存储垃圾”。
# 常用存储方案

### 代码

```
  private void testAndroidFile() {<!-- -->
    Log.d(TAG, "Environment.getDataDirectory() filepath:" + Environment.getDataDirectory());
    Log.d(TAG, "Environment.getDownloadCacheDirectory() filepath:"
        + Environment.getDownloadCacheDirectory());
    Log.d(TAG, "Environment.getRootDirectory() filepath:" + Environment.getRootDirectory());
    Log.d(TAG, "Environment.getExternalStorageDirectory() filepath:"
        + Environment.getExternalStorageDirectory());
    Log.d(TAG, "context.getCacheDir() filepath:" + getCacheDir());
    Log.d(TAG, "context.getDataDir() filepath:" + getDataDir());
    Log.d(TAG, "context.getFilesDir() filepath:" + getFilesDir());
    Log.d(TAG, "context.getObbDir() filepath:" + getObbDir());
    Log.d(TAG, "context.getNoBackupFilesDir() filepath:" + getNoBackupFilesDir());
    Log.d(TAG, "context.getCodeCacheDir() filepath:" + getCodeCacheDir());
    Log.d(TAG, "context.getExternalCacheDir() filepath:" + getExternalCacheDir());
    Log.d(TAG, "context.getExternalFilesDir(null) filepath:" + getExternalFilesDir(null));
  }

```

### 打印结果（手机：小米9、Android10、MIUI 12）

```
Environment.getDataDirectory() filepath:/data
Environment.getDownloadCacheDirectory() filepath:/data/cache
Environment.getRootDirectory() filepath:/system
Environment.getExternalStorageDirectory() filepath:/storage/emulated/0
 context.getCacheDir() filepath:/data/user/0/com.example.androidfiletest/cache
context.getDataDir() filepath:/data/user/0/com.example.androidfiletest
 context.getFilesDir() filepath:/data/user/0/com.example.androidfiletest/files
 context.getObbDir() filepath:/storage/emulated/0/Android/obb/com.example.androidfiletest
context.getNoBackupFilesDir() filepath:/data/user/0/com.example.androidfiletest/no_backup
context.getCodeCacheDir() filepath:/data/user/0/com.example.androidfiletest/code_cache
context.getExternalCacheDir() filepath:/storage/emulated/0/Android/data/com.example.androidfiletest/cache
context.getExternalFilesDir(null) filepath:/storage/emulated/0/Android/data/com.example.androidfiletest/files

```

# Android 11 相关补充

在android 11 上，google官方推荐使用context.getExternalFilesDir(null)来存储应用信息到sdcard。 在此处读写有以下特点：
1. 不需要权限。1. 与私有目录一样，在应用删除之后，该目录下的内容也会被删除。
>  
 在Android 11 的机型上，如果继续使用Environment.getExternalStorageDirectory() 在sdcard根目录操作文件，读写会非常慢。 
 笔者个人操作经验：**删除200MB的文件需要30s左右** 
