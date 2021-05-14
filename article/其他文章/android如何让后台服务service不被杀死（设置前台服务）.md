#android如何让后台服务service不被杀死（设置前台服务）
笔者上篇做了一个定时提醒的小应用，但是最后遗留了一个问题，那就是如果设置提醒的间隔时间过长，那么计时的service便会被android系统kill掉。（主要是由于android自带内存清理）

在进行了大量的查阅和测试后，笔者终于解决了该问题：



当然，在此也要稍微提一下，笔者只测试了，在以一小时为左右的时间内可以不被杀死，还没有测试2个小时以上的情况，更没有测试以天为单位的时间，具体测试如下：（item右下角是程序执行的时间）



<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2620.png" width="400" height="700" alt="">  





好了，进入正题，如何解决service会被kill的情况呢？

那就是设置**前台服务**，那什么是前台服务呢？

如果你希望服务可以一直保持运行状态，而不会由于系统内存不足的原因导致被回收，就可以考虑使用前台服务。前台服务和普通服务最大的区别就在于，它会一直有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看到更加详细的信息，非常类似于通知的效果。



接下来就以墨迹天气为例，如下图：



<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2621.png" alt="">



大家有没有想过，墨迹天气是如何能够在后台不断更新通知栏的中天气，并且不被kill掉的呢？

没错，就是用到了**前台服务**。好了，接下来就讲一下具体怎么使用。



首先看一下整个Service的代码：





```
public class LongRunningService extends Service {


    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        //启用前台服务，主要是startForeground()
        Notification notification = new Notification(R.drawable.queen2, "用电脑时间过长了！白痴！"
                , System.currentTimeMillis());
        notification.setLatestEventInfo(this, "快去休息！！！",
                "一定保护眼睛,不然遗传给孩子，老婆跟别人跑啊。", null);
        //设置通知默认效果
        notification.flags = Notification.FLAG_SHOW_LIGHTS;
        startForeground(1, notification);


        AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
        //读者可以修改此处的Minutes从而改变提醒间隔时间
        //此处是设置每隔55分钟启动一次
        //这是55分钟的毫秒数
        int Minutes = 55 * 60 * 1000;
        //SystemClock.elapsedRealtime()表示1970年1月1日0点至今所经历的时间
        long triggerAtTime = SystemClock.elapsedRealtime() + Minutes;
        //此处设置开启AlarmReceiver这个Service
        Intent i = new Intent(this, AlarmReceiver.class);
        PendingIntent pi = PendingIntent.getBroadcast(this, 0, i, 0);
        //ELAPSED_REALTIME_WAKEUP表示让定时任务的出发时间从系统开机算起，并且会唤醒CPU。
        manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtTime, pi);
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();

        //在Service结束后关闭AlarmManager
        AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
        Intent i = new Intent(this, AlarmReceiver.class);
        PendingIntent pi = PendingIntent.getBroadcast(this, 0, i, 0);
        manager.cancel(pi);

    }
}
```





在笔者的代码中，最核心的部分如下：





```
//启用前台服务，主要是startForeground()
        Notification notification = new Notification(R.drawable.queen2, "用电脑时间过长了！白痴！"
                , System.currentTimeMillis());
        notification.setLatestEventInfo(this, "快去休息！！！",
                "一定保护眼睛,不然遗传给孩子，老婆跟别人跑啊。", null);
        //设置通知默认效果
        notification.flags = Notification.FLAG_SHOW_LIGHTS;
        startForeground(1, notification);
```



此处和通知的使用特别像，但是并没有使用NotificationManager来讲通知显示出来，而是调用了startForeground()方法。调用startForeground()方法后就会让这个Service（在笔者的代码中是LongRunningService）变成一个前台服务了，并且会在系统状态栏显示出来。



可能有部分读者还是不太明白，那么便由笔者再仔细说一下。

在笔者的代码中，除了笔者所说的核心部分外，笔者主要实现了后台计时的功能。倘若笔者不使用前台服务，那么后台计时的服务很可能在运行了几十分钟甚至几分钟的时候就被android系统给回收了。

在设置前台服务后，LongRunningService这个服务成为了前台服务，那么其中实现的功能也是会被系统当做了前台任务运行，并且不会回收，于是便能一直运行了。



当然此方法也是需要慎用，倘若使用了，便会存在android系统不会去自动杀死的一个服务，如果该服务在一直执行，那么手机内存占用和手机耗电量都会自然增加，说不定也会降低用户体验哦。



