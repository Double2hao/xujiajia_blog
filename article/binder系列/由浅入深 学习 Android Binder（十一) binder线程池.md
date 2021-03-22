>Android Binder系列文章：

>[由浅入深 学习 Android Binder（一）- AIDL](https://xujiajia.blog.csdn.net/article/details/109865496)

>[由浅入深 学习 Android Binder（二）- bindService流程](https://xujiajia.blog.csdn.net/article/details/109906012)

>[由浅入深 学习 Android Binder（三）- java binder深究（从java到native）](https://xujiajia.blog.csdn.net/article/details/110730526)

>[由浅入深 学习 Android Binder（四）- ibinderForJavaObject 与 javaObjectForIBinder](https://xujiajia.blog.csdn.net/article/details/111027972)

>[由浅入深 学习 Android Binder（五）- binder如何在进程间传递](https://xujiajia.blog.csdn.net/article/details/111057369)

>[由浅入深 学习 Android Binder（六）- IPC 调用流程](https://xujiajia.blog.csdn.net/article/details/111399789)

>[由浅入深 学习 Android Binder（七）- IServiceManager与ServiceManagerNative（java层）](https://xujiajia.blog.csdn.net/article/details/112131416)

>[由浅入深 学习 Android Binder（八）- IServiceManager与BpServiceManager（native层）](https://xujiajia.blog.csdn.net/article/details/112131416)

>[由浅入深 学习 Android Binder（九）- service_manager 与 svclist](https://xujiajia.blog.csdn.net/article/details/112733698)

>[由浅入深 学习 Android Binder（十）- 总结](https://xujiajia.blog.csdn.net/article/details/112733857)

>[由浅入深 学习 Android Binder（十一) binder线程池](https://xujiajia.blog.csdn.net/article/details/115054785)

# 概述
此篇是补充篇。
“binder线程池”也是binder中一个较常见的知识点，作此文以记之。

# binder线程池的数据结构
刚接触这个知识点最先想到的一定是数据结构。然而，实际上binder线程池并非一个传统的数据结构。
它的大致逻辑如下：
1. 每个进程中只有一个类名为“PoolThread”的数据结构，它继承自"Thread"。
2. binder_driver控制每个进程会启动多少个线程来与binder_driver通信。

PoolThread的源码在ProcessState.cp文件中，源码如下：
```c++
class PoolThread : public Thread
{
public:
    explicit PoolThread(bool isMain)
        : mIsMain(isMain)
    {
    }
    
protected:
    virtual bool threadLoop()
    {
        IPCThreadState::self()->joinThreadPool(mIsMain);
        return false;
    }
    
    const bool mIsMain;
};
```

# binder线程池的启动
启动binder线程池的代码在ProcessState.cp文件中。

startThreadPool中的逻辑如下：
1. 设置了线程池的启动状态
2. 调用了spawnPooledThread方法，传入参数为true。
```c++
void ProcessState::startThreadPool()
{
    AutoMutex _l(mLock);
    if (!mThreadPoolStarted) {
        mThreadPoolStarted = true;
        spawnPooledThread(true);
    }
}
```

# 线程池中线程的启动
spawnPooledThread逻辑如下：
1. 获取binder线程名
2. 创建PoolThread并且run，最终其实会执行到PoolThread的threadLoop()方法，threadLoop()中会执行IPCThreadState的joinThreadPool方法。

```c++
void ProcessState::spawnPooledThread(bool isMain)
{
    if (mThreadPoolStarted) {
        String8 name = makeBinderThreadName();
        ALOGV("Spawning new pooled thread, name=%s\n", name.string());
        sp<Thread> t = new PoolThread(isMain);
        t->run(name.string());
    }
}
```
```c++
class PoolThread : public Thread
{
public:
    explicit PoolThread(bool isMain)
        : mIsMain(isMain)
    {
    }
    
protected:
    virtual bool threadLoop()
    {
        IPCThreadState::self()->joinThreadPool(mIsMain);
        return false;
    }
    
    const bool mIsMain;
};
```
joinThreadPool的逻辑如下：
1. 根据isMain来判断写入哪个cmd。两个cmd与binderDriver通信时分别代表不同的行为。
2. 通过执行getAndExecuteCommand()，将cmd发送给binderDriver，并且在getAndExecuteCommand中会等待binderDriver的返回并处理。

isMain不同值的情况：
- isMain=true。代表是主线程。
BC_ENTER_LOOPER发送给binderDriver后，binderDriver确认能否启动线程，如果不能会报error。
- isMain=false。代表是普通的binder线程。
BC_REGISTER_LOOPER发送给binderDriver后，binderDriver会确认能否启动线程，如果不能会报error。


```c++
void IPCThreadState::joinThreadPool(bool isMain)
{
    LOG_THREADPOOL("**** THREAD %p (PID %d) IS JOINING THE THREAD POOL\n", (void*)pthread_self(), getpid());

    mOut.writeInt32(isMain ? BC_ENTER_LOOPER : BC_REGISTER_LOOPER);

    status_t result;
    mIsLooper = true;
    do {
        processPendingDerefs();
        // now get the next command to be processed, waiting if necessary
        result = getAndExecuteCommand();

        if (result < NO_ERROR && result != TIMED_OUT && result != -ECONNREFUSED && result != -EBADF) {
            ALOGE("getAndExecuteCommand(fd=%d) returned unexpected error %d, aborting",
                  mProcess->mDriverFD, result);
            abort();
        }

        // Let this thread exit the thread pool if it is no longer
        // needed and it is not the main process thread.
        if(result == TIMED_OUT && !isMain) {
            break;
        }
    } while (result != -ECONNREFUSED && result != -EBADF);

    LOG_THREADPOOL("**** THREAD %p (PID %d) IS LEAVING THE THREAD POOL err=%d\n",
        (void*)pthread_self(), getpid(), result);

    mOut.writeInt32(BC_EXIT_LOOPER);
    mIsLooper = false;
    talkWithDriver(false);
}
```

# binderDriver对BC_ENTER_LOOPER的处理

binderDriver对BC_ENTER_LOOPER的处理逻辑如下：
1. 判断是否已经是BINDER_LOOPER_STATE_REGISTERED的状态，即，对应的client进程中的线程，是否是binder主线程。如果是，那么就会报错。
2. 最终会把当前的状态设置成BINDER_LOOPER_STATE_ENTERED，即，对应的client进程中的线程，是非主线程。


```c
static int binder_thread_write(struct binder_proc *proc,
			struct binder_thread *thread,
			binder_uintptr_t binder_buffer, size_t size,
			binder_size_t *consumed)
{
————————————————省略
		case BC_ENTER_LOOPER:
			binder_debug(BINDER_DEBUG_THREADS,
				     "%d:%d BC_ENTER_LOOPER\n",
				     proc->pid, thread->pid);
			if (thread->looper & BINDER_LOOPER_STATE_REGISTERED) {
				thread->looper |= BINDER_LOOPER_STATE_INVALID;
				binder_user_error("%d:%d ERROR: BC_ENTER_LOOPER called after BC_REGISTER_LOOPER\n",
					proc->pid, thread->pid);
			}
			thread->looper |= BINDER_LOOPER_STATE_ENTERED;
			break;
————————————————省略
}
```
# binderDriver对BC_REGISTER_LOOPER的处理
binderDriver对BC_REGISTER_LOOPER的处理的处理逻辑如下：
1. 首先判断下当前是否是BINDER_LOOPER_STATE_ENTERED的状态，即，对应的client进程中的线程，是否是binder主线程，如果是就报错。
2. 判断当前可以请求的线程是否为0，如果没有可以请求的线程，那么就报错。
3. 如果没有报错，那么会把当前“可以请求的线程”减一，当前“开始请求的线程”加一。
并且将状态设置成BINDER_LOOPER_STATE_REGISTERED，即，对应的client进程中的线程，是非主线程。
```c
static int binder_thread_write(struct binder_proc *proc,
			struct binder_thread *thread,
			binder_uintptr_t binder_buffer, size_t size,
			binder_size_t *consumed)
{
————————————————省略
		case BC_REGISTER_LOOPER:
			binder_debug(BINDER_DEBUG_THREADS,
				     "%d:%d BC_REGISTER_LOOPER\n",
				     proc->pid, thread->pid);
			if (thread->looper & BINDER_LOOPER_STATE_ENTERED) {
				thread->looper |= BINDER_LOOPER_STATE_INVALID;
				binder_user_error("%d:%d ERROR: BC_REGISTER_LOOPER called after BC_ENTER_LOOPER\n",
					proc->pid, thread->pid);
			} else if (proc->requested_threads == 0) {
				thread->looper |= BINDER_LOOPER_STATE_INVALID;
				binder_user_error("%d:%d ERROR: BC_REGISTER_LOOPER called without request\n",
					proc->pid, thread->pid);
			} else {
				proc->requested_threads--;
				proc->requested_threads_started++;
			}
			thread->looper |= BINDER_LOOPER_STATE_REGISTERED;
			break;
————————————————省略
}
```

# 何时触发binder线程（非主线程）

binder主线程是在启动binder线程池的时候启动的。
那么非主线程的其他线程在什么时候会被调用呢？

先回到启动binder线程的方法，即ProcessState.cp中的spawnPooledThread():
```c++
void ProcessState::spawnPooledThread(bool isMain)
{
    if (mThreadPoolStarted) {
        String8 name = makeBinderThreadName();
        ALOGV("Spawning new pooled thread, name=%s\n", name.string());
        sp<Thread> t = new PoolThread(isMain);
        t->run(name.string());
    }
}
```

查找的方法非常简单，就是全局搜索调用spawnPooledThread(false)的地方，最终可以发现在IPCThreadState的executeCommand方法中，代码如下：
```c++
status_t IPCThreadState::executeCommand(int32_t cmd)
{
————————————————省略
    case BR_SPAWN_LOOPER:
        mProcess->spawnPooledThread(false);
        break;
————————————————省略
}
```

由此可知还是由binderDriver来通知的，因此在binderDriver的代码中搜索BR_SPAWN_LOOPER，于是可以定位到binder_thread_read()方法。
这个方法中的可以提取的逻辑主要如下：
1. 只有放cmd为BINDER_WORK_TRANSACTION时，才会走到done中的逻辑。否则会直接continue。
2. 在done中一定条件下，会发送BR_SPAWN_LOOPER来通知进程启动binder线程。

done中会执行的条件如下：
1. 当前进程没有可请求的线程，也没有已经ready可用的线程。
2. 当前进程已经开始请求的线程小于最大线程数。（最大线程数后面会探索）
3. 当前线程的状态不能是BINDER_LOOPER_STATE_REGISTERED，也不能是BINDER_LOOPER_STATE_ENTERED。即，对应的client中的线程不能已经启动过。
```c++
static int binder_thread_read(struct binder_proc *proc,
			      struct binder_thread *thread,
			      binder_uintptr_t binder_buffer, size_t size,
			      binder_size_t *consumed, int non_block)
{
————————————————省略

		case BINDER_WORK_TRANSACTION: {
			t = container_of(w, struct binder_transaction, work);
		} break;
————————————————省略
		if (!t)
			continue;
————————————————省略
done:

	*consumed = ptr - buffer;
	if (proc->requested_threads + proc->ready_threads == 0 &&
	    proc->requested_threads_started < proc->max_threads &&
	    (thread->looper & (BINDER_LOOPER_STATE_REGISTERED |
	     BINDER_LOOPER_STATE_ENTERED)) /* the user-space code fails to */
	     /*spawn a new thread if we leave this out */) {
		proc->requested_threads++;
		binder_debug(BINDER_DEBUG_THREADS,
			     "%d:%d BR_SPAWN_LOOPER\n",
			     proc->pid, thread->pid);
		if (put_user(BR_SPAWN_LOOPER, (uint32_t __user *)buffer))
			return -EFAULT;
		binder_stat_br(proc, thread, BR_SPAWN_LOOPER);
	}
	return 0;
}
```

那么问题就继续转移成了“什么时候会调用到BINDER_WORK_TRANSACTION的cmd”。继续在binderDriver的代码中搜索，可以找到binder_transaction（）方法。

```c
static void binder_transaction(struct binder_proc *proc,
			       struct binder_thread *thread,
			       struct binder_transaction_data *tr, int reply)
{
————————————————省略
	t->work.type = BINDER_WORK_TRANSACTION;
	list_add_tail(&t->work.entry, target_list);
	tcomplete->type = BINDER_WORK_TRANSACTION_COMPLETE;
	list_add_tail(&tcomplete->entry, &thread->todo);
	if (target_wait)
		wake_up_interruptible(target_wait);
	return;
————————————————省略
}
```

再继续看下是哪些地方调用了binder_transaction()方法，可以找到binder_thread_write（）中的代码如下。
也就是说当client进程发送BC_TRANSACTION或者BC_REPLY的cmd的时候，会启动binder线程。
```c
static int binder_thread_write(struct binder_proc *proc,
			struct binder_thread *thread,
			binder_uintptr_t binder_buffer, size_t size,
			binder_size_t *consumed)
{
————————————————省略
		case BC_TRANSACTION:
		case BC_REPLY: {
			struct binder_transaction_data tr;

			if (copy_from_user(&tr, ptr, sizeof(tr)))
				return -EFAULT;
			ptr += sizeof(tr);
			binder_transaction(proc, thread, &tr, cmd == BC_REPLY);
			break;
		}
————————————————省略
}
```

那么BC_TRANSACTION与BC_REPLY在client进程中，又分别对应什么操作呢？
在IPCThreadState中分别可以找到对应的方法，代码如下。
分别对应两种场景：
- BC_TRANSACTION：client进程向binderDriver发送IPC调用请求的时候。
- BC_REPLY：client进程收到了binderDriver的IPC调用请求，逻辑执行结束后发送返回值。

> 对binder IPC流程有兴趣的读者可以看下笔者的前文：
> [由浅入深 学习 Android Binder（六）- IPC 调用流程](https://xujiajia.blog.csdn.net/article/details/111399789)
```c++
status_t IPCThreadState::transact(int32_t handle,
                                  uint32_t code, const Parcel& data,
                                  Parcel* reply, uint32_t flags)
{
————————————————省略
    err = writeTransactionData(BC_TRANSACTION_SG, flags, handle, code, data, NULL);
————————————————省略
}
```
```c++
status_t IPCThreadState::sendReply(const Parcel& reply, uint32_t flags)
{
    status_t err;
    status_t statusBuffer;
    err = writeTransactionData(BC_REPLY_SG, flags, -1, 0, reply, &statusBuffer);
    if (err < NO_ERROR) return err;

    return waitForResponse(NULL, NULL);
}
```

# binder线程池大小

前面分析的时候有接触到线程池的最大线程数，即proc->max_threads。
在binderDriver源码中可以搜索到定义这个值的地方，即binder_ioctl()方法：

```c
static long binder_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
————————————————省略
	case BINDER_SET_MAX_THREADS:
		if (copy_from_user(&proc->max_threads, ubuf, sizeof(proc->max_threads))) {
			ret = -EINVAL;
			goto err;
		}
		break;
————————————————省略
}
```

接着全局搜索下BINDER_SET_MAX_THREADS这个cmd，可以在ProcessState中找到open_driver()方法：
```c++
static int open_driver(const char *driver)
{
    int fd = open(driver, O_RDWR | O_CLOEXEC);
    if (fd >= 0) {
        int vers = 0;
        status_t result = ioctl(fd, BINDER_VERSION, &vers);
        if (result == -1) {
            ALOGE("Binder ioctl to obtain version failed: %s", strerror(errno));
            close(fd);
            fd = -1;
        }
        if (result != 0 || vers != BINDER_CURRENT_PROTOCOL_VERSION) {
          ALOGE("Binder driver protocol(%d) does not match user space protocol(%d)! ioctl() return value: %d",
                vers, BINDER_CURRENT_PROTOCOL_VERSION, result);
            close(fd);
            fd = -1;
        }
        size_t maxThreads = DEFAULT_MAX_BINDER_THREADS;
        result = ioctl(fd, BINDER_SET_MAX_THREADS, &maxThreads);
        if (result == -1) {
            ALOGE("Binder ioctl to set max threads failed: %s", strerror(errno));
        }
    } else {
        ALOGW("Opening '%s' failed: %s\n", driver, strerror(errno));
    }
    return fd;
}
```
可以看到默认的binder线程池的大小是DEFAULT_MAX_BINDER_THREADS：
```c++
#define DEFAULT_MAX_BINDER_THREADS 15
```

于是可以得到结论，binder非主线程的最大个数是15，加上主线程的话，总共就是16个线程。

# 总结
binder线程池相关的内容还是挺多的，笔者在此就记录一下个人认为重要的知识点，如对其他细节有兴趣的读者可以自行阅读下源码：
- binder线程池并非一个传统意义上的线程池结构，它在client进程中只有一个继承自Thread的PoolThread类。而线程的启动以及管理都是由binderDriver来控制的。
- binder线程有主线程和非主线程之分，主线程是启动的时候才会有的，每个binder线程池只有一个。其他情况下申请的都是非主线程。
- binder线程池启动的时候，实际上只是启动了client中的binder主线程。
- binder线程(非主线程)有两种情况启动：client进程向binderDriver发送IPC请求，以及 client进程向binderDriver回复IPC请求结果。
- binder线程池的默认大小是16，1个主线程和15个非主线程。
