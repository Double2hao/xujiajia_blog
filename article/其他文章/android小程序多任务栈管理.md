#android小程序多任务栈管理
# 概述

### 微信小程序

要知道什么是小程序的任务栈管理，直接看微信小程序即可：
1. 一次最多启动5个小程序1. 小程序启动到达上限后，会关闭最久未使用的小程序，并且打开新的小程序。
微信小程序使用时，如下图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040281930.png" width="20%" height="20%">

### 实现要点

小程序的任务栈管理，主要有以下要点：
1. 如何启动多个任务栈的小程序1. 如何限制任务栈的启动个数
# Demo

>  
 Demo的github地址如下：  


Demo如下图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040284331.png" width="20%" height="20%">

Demo中打开多任务栈的场景如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040285462.png" width="20%" height="20%">

# 关键实现

### 如何启动多个任务栈的小程序

要点：
1. 让每个界面继承同一个activity1. 所有需要启动的页面注册到一个hashmap中，调度的时候就通过这个hashmap来
```
public class TestActivityOne extends BaseTestActivity{<!-- -->
}

```

```
  private final HashMap&lt;String, TaskStackInfo&gt; mapInfo = new HashMap&lt;&gt;();

  private TaskStackManager() {<!-- -->
    mapInfo.put(TestActivityOne.class.getName(),
        new TaskStackInfo("testprocess1", TestActivityOne.class));
    mapInfo.put(TestActivityTwo.class.getName(),
        new TaskStackInfo("testprocess2", TestActivityTwo.class));
    mapInfo.put(TestActivityThree.class.getName(),
        new TaskStackInfo("testprocess3", TestActivityThree.class));
    mapInfo.put(TestActivityFour.class.getName(),
        new TaskStackInfo("testprocess4", TestActivityFour.class));
  }

```

### 如何限制任务栈的启动个数

要点：
1. 每次启动时，检查任务栈是否到达上限，如果到达上限就需要清理。1. 定义最多启动n个任务栈，实际上最多会存在n+1个。每次启动第n+1个任务栈后，再去清理第一个任务栈。
>  
 如果任务栈到达上限后，先杀死进程，再启动新进程会有两个问题： 
 - 杀死进程需要时间，因此会更耗时。- 实现上会更加麻烦，保证进程被杀死后才能启动新进程。否则可能会出现”新启动的进程马上被杀死“的问题。 


```
  public void startTestActivity(Context context) {<!-- -->
    if (context == null) {<!-- -->
      return;
    }

    ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    if (am == null) {<!-- -->
      return;
    }
    List&lt;String&gt; listCurrentProcess = getCurrentProcessList(am);
    if (listCurrentProcess == null) {<!-- -->
      return;
    }
    chooseActivityToStart(context, listCurrentProcess);

    //如果已经启动了MAX_PROCESS个进程，start时会启动第MAX_PROCESS+1个，并且清理第一个。
    //todo 这里只是demo演示，如果需要更精准的控制，可以在进程启动后通过ipc来调用清理逻辑
    mainHandler.postDelayed(new Runnable() {<!-- -->
      @Override public void run() {<!-- -->
        checkAndClearOldestProcess(context);
      }
    }, 1000);
  }

  private void checkAndClearOldestProcess(Context context) {<!-- -->
    ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    if (am == null) {<!-- -->
      return;
    }
    List&lt;String&gt; listCurrentProcess = getCurrentProcessList(am);
    if (listCurrentProcess == null) {<!-- -->
      return;
    }
    if (listCurrentProcess.size() &gt; MAX_PROCESS) {<!-- -->
      clearOldestProcess(am);
    }
  }

```

# github

>  
 Demo的github地址如下：  
