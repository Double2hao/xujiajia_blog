#由浅入深 学习 Android Binder（六）- IPC 调用流程
>  
 Android Binder系列文章：           


# 概述

如果使用aidl来进行IPC，在client进程执行的transct方法后最终会执行到server进程的onTransact方法，如下图： <img src="https://img-blog.csdnimg.cn/20201121202551112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70#pic_center" alt="在这里插入图片描述">

>  
 对aidl还不是很了解的读者可以看下笔者的前文：  


那么在client进程调用transact后，究竟触发了哪些逻辑呢？本文将针对此点深入探索。

>  
 此文探讨的是两个进程之间的IPC。 如果是同一个进程之间调用，流程会简单一些，有兴趣的读者可以自行探索。 


# 调用流程

先给出流程图，便于读者有一个整体的理解。

>  
 本文会把binder driver当做是一个黑盒，如对binder driver有兴趣，推荐读者看下这篇文章：  


<img src="https://img-blog.csdnimg.cn/20201219073210661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# Client

Client是调用方，调用流程是从java层到native层，最终到binder driver。

### BinderProxy.transact()

BinderProxy定义在Binder.java这个文件中。 这个方法有以下逻辑:
1. 检查Parcel数据是否合法1. 根据开关判断是否需要生成跟踪日志1. 执行transactNative方法，而transactNative则是一个JNI方法。
```
public boolean transact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {<!-- -->
        Binder.checkParcel(this, code, data, "Unreasonably large binder buffer");

        if (mWarnOnBlocking &amp;&amp; ((flags &amp; FLAG_ONEWAY) == 0)) {<!-- -->
            // For now, avoid spamming the log by disabling after we've logged
            // about this interface at least once
            mWarnOnBlocking = false;
            Log.w(Binder.TAG, "Outgoing transactions from this process must be FLAG_ONEWAY",
                    new Throwable());
        }

        final boolean tracingEnabled = Binder.isTracingEnabled();
        if (tracingEnabled) {<!-- -->
            final Throwable tr = new Throwable();
            Binder.getTransactionTracker().addTrace(tr);
            StackTraceElement stackTraceElement = tr.getStackTrace()[1];
            Trace.traceBegin(Trace.TRACE_TAG_ALWAYS,
                    stackTraceElement.getClassName() + "." + stackTraceElement.getMethodName());
        }
        try {<!-- -->
            return transactNative(code, data, reply, flags);
        } finally {<!-- -->
            if (tracingEnabled) {<!-- -->
                Trace.traceEnd(Trace.TRACE_TAG_ALWAYS);
            }
        }
    }
    
    public native boolean transactNative(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException;

```

### android_util_Binder.cpp

在android_util_Binder中可以找到这个jni方法。这个方法是动态注册的。 然后我们直接定位到android_os_BinderProxy_transact这个方法。

```
static const JNINativeMethod gBinderProxyMethods[] = {
     /* name, signature, funcPtr */
——————————————————省略
    {"transactNative",      "(ILandroid/os/Parcel;Landroid/os/Parcel;I)Z", (void*)android_os_BinderProxy_transact},
——————————————————省略
};

```

### android_util_Binder.android_os_BinderProxy_transact

主要逻辑如下：
1. 将java层的Parcel类型数据转化成native层的Parcel1. 通过getBPNativeData() 获取到BpBinder1. 调用BpBinder的tansact方法。
```
static jboolean android_os_BinderProxy_transact(JNIEnv* env, jobject obj,
        jint code, jobject dataObj, jobject replyObj, jint flags) // throws RemoteException
{
    if (dataObj == NULL) {
        jniThrowNullPointerException(env, NULL);
        return JNI_FALSE;
    }

    Parcel* data = parcelForJavaObject(env, dataObj);
    if (data == NULL) {
        return JNI_FALSE;
    }
    Parcel* reply = parcelForJavaObject(env, replyObj);
    if (reply == NULL &amp;&amp; replyObj != NULL) {
        return JNI_FALSE;
    }

    IBinder* target = getBPNativeData(env, obj)-&gt;mObject.get();
    if (target == NULL) {
        jniThrowException(env, "java/lang/IllegalStateException", "Binder has been finalized!");
        return JNI_FALSE;
    }

    ALOGV("Java code calling transact on %p in Java object %p with code %" PRId32 "\n",
            target, obj, code);


    bool time_binder_calls;
    int64_t start_millis;
    if (kEnableBinderSample) {
        // Only log the binder call duration for things on the Java-level main thread.
        // But if we don't
        time_binder_calls = should_time_binder_calls();

        if (time_binder_calls) {
            start_millis = uptimeMillis();
        }
    }

    //printf("Transact from Java code to %p sending: ", target); data-&gt;print();
    status_t err = target-&gt;transact(code, *data, reply, flags);
    //if (reply) printf("Transact from Java code to %p received: ", target); reply-&gt;print();

    if (kEnableBinderSample) {
        if (time_binder_calls) {
            conditionally_log_binder_call(start_millis, target, code);
        }
    }

    if (err == NO_ERROR) {
        return JNI_TRUE;
    } else if (err == UNKNOWN_TRANSACTION) {
        return JNI_FALSE;
    }

    signalExceptionForError(env, obj, err, true /*canThrowRemoteException*/, data-&gt;dataSize());
    return JNI_FALSE;
}

```

### BpBinder.transact()

逻辑很少，先判断BpBinder是否存活，如果存活，就调用IPCThreadState的transact()方法。

```
status_t BpBinder::transact(
    uint32_t code, const Parcel&amp; data, Parcel* reply, uint32_t flags)
{<!-- -->
    // Once a binder has died, it will never come back to life.
    if (mAlive) {<!-- -->
        status_t status = IPCThreadState::self()-&gt;transact(
            mHandle, code, data, reply, flags);
        if (status == DEAD_OBJECT) mAlive = 0;
        return status;
    }

    return DEAD_OBJECT;
}

```

### IPCThreadState.transact()

主要逻辑如下：
1. 通过writeTransactionData将需要IPC的数据转化成binder driver 通信的形式。1. 调用waitForResponse().
```
status_t IPCThreadState::transact(int32_t handle,
                                  uint32_t code, const Parcel&amp; data,
                                  Parcel* reply, uint32_t flags)
{
    status_t err;

    flags |= TF_ACCEPT_FDS;

    IF_LOG_TRANSACTIONS() {
        alog &lt;&lt; "BC_TRANSACTION thr " &lt;&lt; (void*)pthread_self() &lt;&lt; " / hand "
            &lt;&lt; handle &lt;&lt; " / code " &lt;&lt; TypeCode(code) &lt;&lt; ": "
            &lt;&lt; indent &lt;&lt; data &lt;&lt; dedent &lt;&lt; endl;
    }

    LOG_ONEWAY("&gt;&gt;&gt;&gt; SEND from pid %d uid %d %s", getpid(), getuid(),
        (flags &amp; TF_ONE_WAY) == 0 ? "READ REPLY" : "ONE WAY");
    err = writeTransactionData(BC_TRANSACTION_SG, flags, handle, code, data, NULL);

    if (err != NO_ERROR) {
        if (reply) reply-&gt;setError(err);
        return (mLastError = err);
    }

    if ((flags &amp; TF_ONE_WAY) == 0) {
        #if 0
        if (code == 4) { // relayout
            ALOGI("&gt;&gt;&gt;&gt;&gt;&gt; CALLING transaction 4");
        } else {
            ALOGI("&gt;&gt;&gt;&gt;&gt;&gt; CALLING transaction %d", code);
        }
        #endif
        if (reply) {
            err = waitForResponse(reply);
        } else {
            Parcel fakeReply;
            err = waitForResponse(&amp;fakeReply);
        }
        #if 0
        if (code == 4) { // relayout
            ALOGI("&lt;&lt;&lt;&lt;&lt;&lt; RETURNING transaction 4");
        } else {
            ALOGI("&lt;&lt;&lt;&lt;&lt;&lt; RETURNING transaction %d", code);
        }
        #endif

        IF_LOG_TRANSACTIONS() {
            alog &lt;&lt; "BR_REPLY thr " &lt;&lt; (void*)pthread_self() &lt;&lt; " / hand "
                &lt;&lt; handle &lt;&lt; ": ";
            if (reply) alog &lt;&lt; indent &lt;&lt; *reply &lt;&lt; dedent &lt;&lt; endl;
            else alog &lt;&lt; "(none requested)" &lt;&lt; endl;
        }
    } else {
        err = waitForResponse(NULL, NULL);
    }

    return err;
}

```

### IPCThreadState.writeTransactionData()

主要逻辑如下：
1. 将Parcel中的数据转化成binder_transaction_data_sg对象1. 将binder_transaction_data_sg对象写入 mOut。(在.h文件中可以看到mOut也是一个Parcel对象)
```
status_t IPCThreadState::writeTransactionData(int32_t cmd, uint32_t binderFlags,
    int32_t handle, uint32_t code, const Parcel&amp; data, status_t* statusBuffer)
{
    binder_transaction_data_sg tr_sg;
    /* Don't pass uninitialized stack data to a remote process */
    tr_sg.transaction_data.target.ptr = 0;
    tr_sg.transaction_data.target.handle = handle;
    tr_sg.transaction_data.code = code;
    tr_sg.transaction_data.flags = binderFlags;
    tr_sg.transaction_data.cookie = 0;
    tr_sg.transaction_data.sender_pid = 0;
    tr_sg.transaction_data.sender_euid = 0;

    const status_t err = data.errorCheck();
    if (err == NO_ERROR) {
        tr_sg.transaction_data.data_size = data.ipcDataSize();
        tr_sg.transaction_data.data.ptr.buffer = data.ipcData();
        tr_sg.transaction_data.offsets_size = data.ipcObjectsCount()*sizeof(binder_size_t);
        tr_sg.transaction_data.data.ptr.offsets = data.ipcObjects();
        tr_sg.buffers_size = data.ipcBufferSize();
    } else if (statusBuffer) {
        tr_sg.transaction_data.flags |= TF_STATUS_CODE;
        *statusBuffer = err;
        tr_sg.transaction_data.data_size = sizeof(status_t);
        tr_sg.transaction_data.data.ptr.buffer = reinterpret_cast&lt;uintptr_t&gt;(statusBuffer);
        tr_sg.transaction_data.offsets_size = 0;
        tr_sg.transaction_data.data.ptr.offsets = 0;
        tr_sg.buffers_size = 0;
    } else {
        return (mLastError = err);
    }

    mOut.writeInt32(cmd);
    mOut.write(&amp;tr_sg, sizeof(tr_sg));

    return NO_ERROR;
}

```

### IPCThreadState.waitForResponse

主要逻辑如下：
1. waitForResponse中有一个while的死循环，只有当收到消息后才会退出循环。1. 先会调用talkWithDriver与binder driver通信，数据会通过mIn来接收，mIn也是一个Parcel对象。1. 通过mIn获取到cmd，根据cmd来区分不同的通信逻辑。如果是正常结束的话，会直接走到BR_TRANSACTION_COMPLETE的逻辑中。通过goto到finish的位置，如果没有错误就会直接return。
```
status_t IPCThreadState::waitForResponse(Parcel *reply, status_t *acquireResult)
{
    uint32_t cmd;
    int32_t err;

    while (1) {
        if ((err=talkWithDriver()) &lt; NO_ERROR) break;
        err = mIn.errorCheck();
        if (err &lt; NO_ERROR) break;
        if (mIn.dataAvail() == 0) continue;

        cmd = (uint32_t)mIn.readInt32();

        IF_LOG_COMMANDS() {
            alog &lt;&lt; "Processing waitForResponse Command: "
                &lt;&lt; getReturnString(cmd) &lt;&lt; endl;
        }

        switch (cmd) {
        case BR_TRANSACTION_COMPLETE:
            if (!reply &amp;&amp; !acquireResult) goto finish;
            break;

        case BR_DEAD_REPLY:
            err = DEAD_OBJECT;
            goto finish;

        case BR_FAILED_REPLY:
            err = FAILED_TRANSACTION;
            goto finish;

        case BR_ACQUIRE_RESULT:
            {
                ALOG_ASSERT(acquireResult != NULL, "Unexpected brACQUIRE_RESULT");
                const int32_t result = mIn.readInt32();
                if (!acquireResult) continue;
                *acquireResult = result ? NO_ERROR : INVALID_OPERATION;
            }
            goto finish;

        case BR_REPLY:
            {
                binder_transaction_data tr;
                err = mIn.read(&amp;tr, sizeof(tr));
                ALOG_ASSERT(err == NO_ERROR, "Not enough command data for brREPLY");
                if (err != NO_ERROR) goto finish;

                if (reply) {
                    if ((tr.flags &amp; TF_STATUS_CODE) == 0) {
                        reply-&gt;ipcSetDataReference(
                            reinterpret_cast&lt;const uint8_t*&gt;(tr.data.ptr.buffer),
                            tr.data_size,
                            reinterpret_cast&lt;const binder_size_t*&gt;(tr.data.ptr.offsets),
                            tr.offsets_size/sizeof(binder_size_t),
                            freeBuffer, this);
                    } else {
                        err = *reinterpret_cast&lt;const status_t*&gt;(tr.data.ptr.buffer);
                        freeBuffer(NULL,
                            reinterpret_cast&lt;const uint8_t*&gt;(tr.data.ptr.buffer),
                            tr.data_size,
                            reinterpret_cast&lt;const binder_size_t*&gt;(tr.data.ptr.offsets),
                            tr.offsets_size/sizeof(binder_size_t), this);
                    }
                } else {
                    freeBuffer(NULL,
                        reinterpret_cast&lt;const uint8_t*&gt;(tr.data.ptr.buffer),
                        tr.data_size,
                        reinterpret_cast&lt;const binder_size_t*&gt;(tr.data.ptr.offsets),
                        tr.offsets_size/sizeof(binder_size_t), this);
                    continue;
                }
            }
            goto finish;

        default:
            err = executeCommand(cmd);
            if (err != NO_ERROR) goto finish;
            break;
        }
    }

finish:
    if (err != NO_ERROR) {
        if (acquireResult) *acquireResult = err;
        if (reply) reply-&gt;setError(err);
        mLastError = err;
    }

    return err;
}

```

### IPCThreadState.talkWithDriver

主要逻辑如下：
1. 先从mOut对象中读取需要传输的数据。1. 通过ioctl方法来与binder driver通信。1. 通信结束后，使用mIn对象来接收数据。
```
status_t IPCThreadState::talkWithDriver(bool doReceive)
{
    if (mProcess-&gt;mDriverFD &lt;= 0) {
        return -EBADF;
    }

    binder_write_read bwr;

    // Is the read buffer empty?
    const bool needRead = mIn.dataPosition() &gt;= mIn.dataSize();

    // We don't want to write anything if we are still reading
    // from data left in the input buffer and the caller
    // has requested to read the next data.
    const size_t outAvail = (!doReceive || needRead) ? mOut.dataSize() : 0;

    bwr.write_size = outAvail;
    bwr.write_buffer = (uintptr_t)mOut.data();

    // This is what we'll read.
    if (doReceive &amp;&amp; needRead) {
        bwr.read_size = mIn.dataCapacity();
        bwr.read_buffer = (uintptr_t)mIn.data();
    } else {
        bwr.read_size = 0;
        bwr.read_buffer = 0;
    }

    IF_LOG_COMMANDS() {
        if (outAvail != 0) {
            alog &lt;&lt; "Sending commands to driver: " &lt;&lt; indent;
            const void* cmds = (const void*)bwr.write_buffer;
            const void* end = ((const uint8_t*)cmds)+bwr.write_size;
            alog &lt;&lt; HexDump(cmds, bwr.write_size) &lt;&lt; endl;
            while (cmds &lt; end) cmds = printCommand(alog, cmds);
            alog &lt;&lt; dedent;
        }
        alog &lt;&lt; "Size of receive buffer: " &lt;&lt; bwr.read_size
            &lt;&lt; ", needRead: " &lt;&lt; needRead &lt;&lt; ", doReceive: " &lt;&lt; doReceive &lt;&lt; endl;
    }

    // Return immediately if there is nothing to do.
    if ((bwr.write_size == 0) &amp;&amp; (bwr.read_size == 0)) return NO_ERROR;

    bwr.write_consumed = 0;
    bwr.read_consumed = 0;
    status_t err;
    do {
        IF_LOG_COMMANDS() {
            alog &lt;&lt; "About to read/write, write size = " &lt;&lt; mOut.dataSize() &lt;&lt; endl;
        }
#if defined(__ANDROID__)
        if (ioctl(mProcess-&gt;mDriverFD, BINDER_WRITE_READ, &amp;bwr) &gt;= 0)
            err = NO_ERROR;
        else
            err = -errno;
#else
        err = INVALID_OPERATION;
#endif
        if (mProcess-&gt;mDriverFD &lt;= 0) {
            err = -EBADF;
        }
        IF_LOG_COMMANDS() {
            alog &lt;&lt; "Finished read/write, write size = " &lt;&lt; mOut.dataSize() &lt;&lt; endl;
        }
    } while (err == -EINTR);

    IF_LOG_COMMANDS() {
        alog &lt;&lt; "Our err: " &lt;&lt; (void*)(intptr_t)err &lt;&lt; ", write consumed: "
            &lt;&lt; bwr.write_consumed &lt;&lt; " (of " &lt;&lt; mOut.dataSize()
                        &lt;&lt; "), read consumed: " &lt;&lt; bwr.read_consumed &lt;&lt; endl;
    }

    if (err &gt;= NO_ERROR) {
        if (bwr.write_consumed &gt; 0) {
            if (bwr.write_consumed &lt; mOut.dataSize())
                mOut.remove(0, bwr.write_consumed);
            else {
                mOut.setDataSize(0);
                processPostWriteDerefs();
            }
        }
        if (bwr.read_consumed &gt; 0) {
            mIn.setDataSize(bwr.read_consumed);
            mIn.setDataPosition(0);
        }
        IF_LOG_COMMANDS() {
            alog &lt;&lt; "Remaining data size: " &lt;&lt; mOut.dataSize() &lt;&lt; endl;
            alog &lt;&lt; "Received commands from driver: " &lt;&lt; indent;
            const void* cmds = mIn.data();
            const void* end = mIn.data() + mIn.dataSize();
            alog &lt;&lt; HexDump(cmds, mIn.dataSize()) &lt;&lt; endl;
            while (cmds &lt; end) cmds = printReturnCommand(alog, cmds);
            alog &lt;&lt; dedent;
        }
        return NO_ERROR;
    }

    return err;
}

```

# Server

Server是被调用方，因此与Client的流程正好想法，先从binder driver到native层，最后到java层。

### IPCThreadState.waitForResponse

waitForResponse前面在client中已经分析过，此处就放最关键的代码。 主要逻辑：
1. waitForResponse中有一个while的死循环，只有当收到消息后才会退出循环。1. server接受到数据后，会进入到defalut的逻辑中，最终会执行到executeCommand方法。
```

status_t IPCThreadState::waitForResponse(Parcel *reply, status_t *acquireResult)
{
——————————————————省略
        default:
            err = executeCommand(cmd);
            if (err != NO_ERROR) goto finish;
            break;
        }
——————————————————省略
}

```

### IPCThreadState.executeCommand

此方法中的逻辑有很多，我们就直接看下server被调用时的BR_TRANSACTION的逻辑：
1. 通过mIn对象来获取binder driver传来的数据1. 在前面的文章中，我们了解到，如果binder是本地进程时，binder_transaction_data.cookie其实存储的是JavaBBinder对象。此处是server进程接收到client的远程调用在本地进程调用自己的逻辑，因此最终执行到的是JavaBBinder.tansact逻辑。
>  
 前文地址：   


```
status_t IPCThreadState::executeCommand(int32_t cmd)
{
    BHwBinder* obj;
    RefBase::weakref_type* refs;
    status_t result = NO_ERROR;
    switch ((uint32_t)cmd) {
——————————————————省略
        case BR_TRANSACTION:
        {
            binder_transaction_data tr;
            result = mIn.read(&amp;tr, sizeof(tr));
            ALOG_ASSERT(result == NO_ERROR,
                "Not enough command data for brTRANSACTION");
            if (result != NO_ERROR) break;

            Parcel buffer;
            buffer.ipcSetDataReference(
                reinterpret_cast&lt;const uint8_t*&gt;(tr.data.ptr.buffer),
                tr.data_size,
                reinterpret_cast&lt;const binder_size_t*&gt;(tr.data.ptr.offsets),
                tr.offsets_size/sizeof(binder_size_t), freeBuffer, this);

            const pid_t origPid = mCallingPid;
            const uid_t origUid = mCallingUid;
            const int32_t origStrictModePolicy = mStrictModePolicy;
            const int32_t origTransactionBinderFlags = mLastTransactionBinderFlags;

            mCallingPid = tr.sender_pid;
            mCallingUid = tr.sender_euid;
            mLastTransactionBinderFlags = tr.flags;

            //ALOGI("&gt;&gt;&gt;&gt; TRANSACT from pid %d uid %d\n", mCallingPid, mCallingUid);

            Parcel reply;
            status_t error;
            bool reply_sent = false;
            IF_LOG_TRANSACTIONS() {
                alog &lt;&lt; "BR_TRANSACTION thr " &lt;&lt; (void*)pthread_self()
                    &lt;&lt; " / obj " &lt;&lt; tr.target.ptr &lt;&lt; " / code "
                    &lt;&lt; TypeCode(tr.code) &lt;&lt; ": " &lt;&lt; indent &lt;&lt; buffer
                    &lt;&lt; dedent &lt;&lt; endl
                    &lt;&lt; "Data addr = "
                    &lt;&lt; reinterpret_cast&lt;const uint8_t*&gt;(tr.data.ptr.buffer)
                    &lt;&lt; ", offsets addr="
                    &lt;&lt; reinterpret_cast&lt;const size_t*&gt;(tr.data.ptr.offsets) &lt;&lt; endl;
            }

            auto reply_callback = [&amp;] (auto &amp;replyParcel) {
                if (reply_sent) {
                    // Reply was sent earlier, ignore it.
                    ALOGE("Dropping binder reply, it was sent already.");
                    return;
                }
                reply_sent = true;
                if ((tr.flags &amp; TF_ONE_WAY) == 0) {
                    replyParcel.setError(NO_ERROR);
                    sendReply(replyParcel, 0);
                } else {
                    ALOGE("Not sending reply in one-way transaction");
                }
            };

            if (tr.target.ptr) {
                // We only have a weak reference on the target object, so we must first try to
                // safely acquire a strong reference before doing anything else with it.
                if (reinterpret_cast&lt;RefBase::weakref_type*&gt;(
                        tr.target.ptr)-&gt;attemptIncStrong(this)) {
                    error = reinterpret_cast&lt;BHwBinder*&gt;(tr.cookie)-&gt;transact(tr.code, buffer,
                            &amp;reply, tr.flags, reply_callback);
                    reinterpret_cast&lt;BHwBinder*&gt;(tr.cookie)-&gt;decStrong(this);
                } else {
                    error = UNKNOWN_TRANSACTION;
                }

            } else {
                error = mContextObject-&gt;transact(tr.code, buffer, &amp;reply, tr.flags, reply_callback);
            }

            if ((tr.flags &amp; TF_ONE_WAY) == 0) {
                if (!reply_sent) {
                    // Should have been a reply but there wasn't, so there
                    // must have been an error instead.
                    reply.setError(error);
                    sendReply(reply, 0);
                } else {
                    if (error != NO_ERROR) {
                        ALOGE("transact() returned error after sending reply.");
                    } else {
                        // Ok, reply sent and transact didn't return an error.
                    }
                }
            } else {
                // One-way transaction, don't care about return value or reply.
            }

            //ALOGI("&lt;&lt;&lt;&lt; TRANSACT from pid %d restore pid %d uid %d\n",
            //     mCallingPid, origPid, origUid);


            mCallingPid = origPid;
            mCallingUid = origUid;
            mStrictModePolicy = origStrictModePolicy;
            mLastTransactionBinderFlags = origTransactionBinderFlags;

            IF_LOG_TRANSACTIONS() {
                alog &lt;&lt; "BC_REPLY thr " &lt;&lt; (void*)pthread_self() &lt;&lt; " / obj "
                    &lt;&lt; tr.target.ptr &lt;&lt; ": " &lt;&lt; indent &lt;&lt; reply &lt;&lt; dedent &lt;&lt; endl;
            }

        }
        break;
——————————————————省略
        }
——————————————————省略    
}
    

```

### BBinder.transact

JavaBBinder是BBinder的子类，由于JavaBBinder中没有重新实现transact方法，因此实际上调用的是父类BBinder的transact方法。 transact中的其实就只是调用了自己的onTransact方法，由于这个对象的类型本身是JavaBBinder，因此要看下JavaBBinder的onTransact。

```
status_t BBinder::transact(
    uint32_t code, const Parcel&amp; data, Parcel* reply, uint32_t flags)
{
    data.setDataPosition(0);

    status_t err = NO_ERROR;
    switch (code) {
        case PING_TRANSACTION:
            reply-&gt;writeInt32(pingBinder());
            break;
        default:
            err = onTransact(code, data, reply, flags);
            break;
    }

    if (reply != NULL) {
        reply-&gt;setDataPosition(0);
    }

    return err;
}

```

### JavaBBinder.onTransact

onTransact的主要逻辑其实就是通过JNI来调用java层Binder的execTransact方法。

>  
 对JNI调用以及gBinderOffsets有兴趣的读者可以看下笔者前面的文章：  


```
virtual status_t onTransact(
        uint32_t code, const Parcel&amp; data, Parcel* reply, uint32_t flags = 0)
    {
——————————————————省略
        jboolean res = env-&gt;CallBooleanMethod(mObject, gBinderOffsets.mExecTransact,
            code, reinterpret_cast&lt;jlong&gt;(&amp;data), reinterpret_cast&lt;jlong&gt;(reply), flags);

——————————————————省略
    }

```

```
static int int_register_android_os_Binder(JNIEnv* env)
{
    jclass clazz = FindClassOrDie(env, kBinderPathName);

    gBinderOffsets.mClass = MakeGlobalRefOrDie(env, clazz);
    gBinderOffsets.mExecTransact = GetMethodIDOrDie(env, clazz, "execTransact", "(IJJI)Z");
    gBinderOffsets.mObject = GetFieldIDOrDie(env, clazz, "mObject", "J");

    return RegisterMethodsOrDie(
        env, kBinderPathName,
        gBinderMethods, NELEM(gBinderMethods));
}

```

### Binder.execTransasct

这里注释可以看到，JNI的逻辑是在android_util_Binder.cpp这个文件中，也就是指我们前面看的JavaBBinder的相关逻辑。 这个方法最终执行到的是execTransactInternal方法，我们继续看。

```
    // Entry point from android_util_Binder.cpp's onTransact
    @UnsupportedAppUsage
    private boolean execTransact(int code, long dataObj, long replyObj,
            int flags) {<!-- -->
        // At that point, the parcel request headers haven't been parsed so we do not know what
        // WorkSource the caller has set. Use calling uid as the default.
        final int callingUid = Binder.getCallingUid();
        final long origWorkSource = ThreadLocalWorkSource.setUid(callingUid);
        try {<!-- -->
            return execTransactInternal(code, dataObj, replyObj, flags, callingUid);
        } finally {<!-- -->
            ThreadLocalWorkSource.restore(origWorkSource);
        }
    }

```

### Binder.execTransactInternal

主要逻辑如下：
1. 通过Parcel.obtain()将native层的Parcel数据转化成java层的的Parcel对象。1. 调用Binder.onTransact方法，如果这个Binder通过aidl生成的，那么就是Stub的onTransact方法。
```
    private boolean execTransactInternal(int code, long dataObj, long replyObj, int flags,
            int callingUid) {<!-- -->
——————————————————省略
        Parcel data = Parcel.obtain(dataObj);
        Parcel reply = Parcel.obtain(replyObj);
——————————————————省略
        try {<!-- -->
——————————————————省略
            res = onTransact(code, data, reply, flags);
        } 
——————————————————省略
        return res;
    }

```

### Binder.onTransact

我们此处直接看下笔者demo中的Stub.onTransact方法的实现,主要逻辑如下：
1. 从data中取出参数1. 把参数传入IPC方法中执行。1. 将执行结果通过reply，也就是Parcel对象写回去。
>  
 对demo有兴趣，或者对aidl不了解读者可以看下笔者的前文：  


```
    @Override
    public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags)
        throws android.os.RemoteException {<!-- -->
      java.lang.String descriptor = DESCRIPTOR;
      switch (code) {<!-- -->
        case INTERFACE_TRANSACTION: {<!-- -->
          reply.writeString(descriptor);
          return true;
        }
        case TRANSACTION_testFunction: {<!-- -->
          data.enforceInterface(descriptor);//需要传递DESCRIPTOR，到另一个进程可以找到对应的Binder
          java.lang.String _arg0;
          _arg0 = data.readString();//读取参数
          int _result = this.testFunction(_arg0);//运行
          reply.writeNoException();
          reply.writeInt(_result);//写入返回值
          return true;
        }
        default: {<!-- -->
          return super.onTransact(code, data, reply, flags);
        }
      }
    }

```

# 总结

本文从java层到native层梳理了一遍client与server一次IPC的调用流程。 笔者并不期望用一篇文章就能将整个Binder说明白，本文更多的只是提供一个理解Binder的思路，另外就是让读者对binder ipc的流程有一个大概的理解。

在整个调用的流程中，涉及到的很多知识点，本文都是没有具体深入的。 比如java层的Binder和BinderProxy，native层的JavaBBinder和BinderProxy，还有native层的ProcessState和IPCThreadState等。 如果对其中某个知识点有兴趣的读者可以自行探索下，或者可以参考下此系列的其他文章。 希望这篇文章能对你有所帮助。

<img src="https://img-blog.csdnimg.cn/20201219073210661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">
