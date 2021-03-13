#【LiteApp系列】埋点的设计
# 前言

埋点是线上项目采集数据的一种重要方式，这些可能是对error数据的采集，也可能是对用户偏好的采集。其实质其实就是在代码的相应位置添加埋点，执行相应埋点代码后能够将信息记录到对应文件中，然后在一定时机下上传到服务器。

# 关键问题

埋点要解决的几个关键问题如下：  1、由于全局都可能使用到埋点，因此一般埋点为单例模式。  2、放置埋点不可避免在多个线程的情况，因此要使用handler来实现，埋点要有自己的线程并且有Looper循环。  3、部分埋点逻辑可能要由业务中去实现，因此要由相应接口。

# 代码

1、定义了logTypeFilter变量，来保证只有已经定义的埋点类型才会被记录。未知的（没有定义的）埋点类型会被过滤。  2、Record内部类是此处定义的埋点结构。  3、此处record(String logType, String logContent, Exception exception)是提供给全局使用的设置埋点的方法。

```
/**
 *
 * Copyright 2018 iQIYI.com
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 *
 */
package com.iqiyi.halberd.liteapp.utils;

import android.annotation.SuppressLint;
import android.content.Context;
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.os.Process;
import android.util.Log;

/**
 * Created by eggizhang@qiyi.com on 17-10-27.
 * Using this utils for performance recording and process recording of lite app
 */
public class LogUtils {<!-- -->
    public static final String TAG = LogUtils.class.getName();

    public static final String LOG_MINI_PROGRAM_ACTIVITY_LIFE_CYCLE = "log_type_app_activity_life_cycle";
    public static final String LOG_MINI_PROGRAM_PAGE_LIFE_CYCLE = "log_type_app_page_life_cycle";
    public static final String LOG_MINI_PROGRAM_FRAGMENT_LIFE_CYCLE = "log_type_app_fragment_life_cycle";
    public static final String LOG_MINI_PROGRAM_CACHE = "log_lite_app_cache";
    public static final String LOG_MINI_PROGRAM_EVENT = "log_lite_app_event";
    public static final String LOG_MINI_PROGRAM_ERROR = "log_lite_app_error";

    public static final String ACTION_START=" action start";
    public static final String ACTION_STOP=" action stop";

    public static final String LIFE_OBJECT_CREATE = " object create";
    public static final String LIFE_CREATE = " onCreate";
    public static final String LIFE_START = " onStart";
    public static final String LIFE_RESUME = " onResume";
    public static final String LIFE_PAUSE = " onPause";
    public static final String LIFE_STOP = " onStop";
    public static final String LIFE_DESTROY = " onDestroy";
    public static final String LIFE_CREATE_VIEW = " onCreateView";
    public static final String LIFE_DESTROY_VIEW = " onDestroyView";

    public static final String LIFE_ACTIVITY = " liteAppActivity";
    public static final String LIFE_FRAGMENT = " liteAppFragment";
    public static final String LIFE_PAGE = " liteAppPage";
    public static final String LIFE_WEB_VIEW = " halWebView";

    public static final String PAGE_CONTEXT_CREATE = " page context create";
    public static final String PAGE_CONTEXT_DISPOSE = " page context dispose";
    public static final String PAGE_THREAD_CREATE = " page thread create";
    public static final String PAGE_THREAD_STOP = " page thread stop";

    public static final String FRAGMENT_PUSH = " fragment push";
    public static final String FRAGMENT_POP = " fragment pop";

    public static final String CACHE_PAGE = " page cache";
    public static final String CACHE_FILE = " file cache";
    public static final String CACHE_IMAGE = " image cache";

    public static final String CACHE_MEMORY_HIT = " memory hit";
    public static final String CACHE_DISK_HIT = " disk hit";
    public static final String CACHE_MEMORY_CREATE = " memory create";
    public static final String CACHE_DISK_CREATE = " disk create";
    public static final String CACHE_NETWORK = " network access";
    public static final String CACHE_CHECK_UPDATE = "check update";

    public static final String COMMON_FAIL = "failed";
    public static final String COMMON_SUCCESS = "success";

    public static final String EVENT_TYPE = " event type is ";

    private final static String[] logTypeFilter = new String[]{
            LOG_MINI_PROGRAM_ACTIVITY_LIFE_CYCLE,
            LOG_MINI_PROGRAM_PAGE_LIFE_CYCLE,
            LOG_MINI_PROGRAM_FRAGMENT_LIFE_CYCLE,
            LOG_MINI_PROGRAM_CACHE,
            LOG_MINI_PROGRAM_EVENT,
            LOG_MINI_PROGRAM_ERROR
    };

    private static LogUtils instance = null;
    private String applicationFilesDir=null;
    private Thread logThread = null;
    private String manufacture = null;
    private String model = null;
    private String liteAppID = null;

    private boolean isNormalSize = true;
    private int messageQueueSize=0;
    private static final int QUEUE_MAX_SIZE = 100;
    private static final int QUEUE_NORMAL_SIZE = 5;

    private Handler handler;
    private static final int LOG_RECORD=0;
    private static final int REMOVE_EXTRA_FILES=1;

    private LogProvider logProvider;

    public static void setLogProvider(LogProvider logProvider) {
        getInstance().logProvider = logProvider;
    }

    public interface LogProvider {<!-- -->
        void doRecord(Record record, Exception e);
        void clearLogFile(String applicationFileDir);
    }

    private LogUtils() {
    }

    public static LogUtils getInstance() {
        if (instance == null) {
            synchronized (LogUtils.class) {
                if (instance == null) {
                    instance = new LogUtils();
                }
            }
        }
        return instance;
    }

    public void init(Context context) {
        if (logThread == null) {
            applicationFilesDir=context.getFilesDir().getPath();
            manufacture = android.os.Build.MANUFACTURER;
            model = android.os.Build.MODEL;
            startThreadLoop();
        }
    }

    public String getApplicationFilesDir() {
        return applicationFilesDir;
    }

    public void setLiteAppIdIfNull(String liteAppID) {
        if (this.liteAppID == null) {
            this.liteAppID = liteAppID;
        }
    }

    private void startThreadLoop() {
        logThread = new Thread(new Runnable() {
            @SuppressLint("HandlerLeak")
            @Override
            public void run() {
                Looper.prepare();
                handler=new Handler(){
                    @Override
                    public void handleMessage(Message msg) {
                        super.handleMessage(msg);
                        try{
                            switch (msg.what){
                                case LOG_RECORD:
                                    consume((Record) msg.obj);
                                    break;
                                case REMOVE_EXTRA_FILES:
                                    if(logProvider!=null)
                                        logProvider.clearLogFile(applicationFilesDir);
                                    break;
                            }
                        } catch (Exception e) {
                            Log.e(LOG_MINI_PROGRAM_ERROR, "logFile name unknown error.", e);
                        }
                    }
                };

                Message msg=handler.obtainMessage();
                msg.what=REMOVE_EXTRA_FILES;
                handler.sendMessage(msg);

                Looper.loop();
            }
        });
        logThread.start();
    }

    private void consume(Record record) {
        //TODO 这里留上传接口
        if (record.exception == null) {
            Log.v("lite-app-log", record.toString());
        } else {
            Log.e("lite-app-error", record.toString(), record.exception);
        }
        if(logProvider!=null)
            logProvider.doRecord(record, record.exception);
        messageQueueSize--;
    }

    public class Record {<!-- -->
        private String logType;
        private long time;
        private int pid;
        private long tid;
        private String content;
        private Exception exception = null;

        public String getContent() {
            return content;
        }

        public long getTime() {
            return time;
        }

        @Override
        public String toString() {
            if (messageQueueSize &gt; QUEUE_MAX_SIZE) {
                isNormalSize = false;
            } else if (messageQueueSize &lt; QUEUE_NORMAL_SIZE) {
                isNormalSize = true;
            }

            if (isNormalSize) {
                return "time:" + time + "," +
                        "content:" + content + "," +
                        "manufacture:" + manufacture + "," +
                        "model:" + model + "," +
                        "pid:" + pid + "," +
                        "tid:" + tid + "," +
                        "queueing:" + messageQueueSize + "," +
                        "logType:" + logType + "," +
                        "liteAppID:" + liteAppID + "," +
                        ".";
            } else {
                return "too muck log" + logType;
            }
        }
    }

    public static void log(String logType, String logContent) {
        if(getInstance().logThread!=null&amp;&amp;getInstance().handler!=null)
            getInstance().record(logType, logContent, null);
    }

    public static void logError(String logType, String logContent, Exception exception) {
        if(getInstance().logThread!=null&amp;&amp;getInstance().handler!=null)
            getInstance().record(logType, logContent, exception);
    }

    private void record(String logType, String logContent, Exception exception) {
        for (String aLogTypeFilter : logTypeFilter) {
            if (aLogTypeFilter.equals(logType)) {
                Record newRecord = new Record();
                newRecord.time = System.currentTimeMillis();
                newRecord.pid = Process.myPid();
                newRecord.tid = Thread.currentThread().getId();
                newRecord.content = logContent;
                newRecord.logType = logType;

                if (exception != null &amp;&amp; logType.equals(LOG_MINI_PROGRAM_ERROR)) {
                    newRecord.exception = exception;
                }

                Message msg=handler.obtainMessage();
                msg.what=LOG_RECORD;
                msg.obj=newRecord;
                handler.sendMessage(msg);
                messageQueueSize++;
                return;
            }
        }
    }
}

```

>  
 更多内容可以查看github官方开源项目：   
