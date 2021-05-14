#由浅入深 学习 Android Binder（九）- service_manager 与 svclist
>  
 Android Binder系列文章：            


# 概述

前文讲过了java层与native层的IServiceManager，两者最终其实都是通过transact方法 走到了binder driver。 之后binder driver便会把工作交给service_manager来处理，这个类的path是/frameworks/native/cmds/servicemanager/service_manager.c。

>  
 前文地址：   


为了方便读者有个整体的认知，附上整体的流程图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910646000.png " alt="在这里插入图片描述">

# 存储结构svclist

```
struct svcinfo
{<!-- -->
    struct svcinfo *next;
    uint32_t handle;
    struct binder_death death;
    int allow_isolated;
    uint32_t dumpsys_priority;
    size_t len;
    uint16_t name[0];
};

struct svcinfo *svclist = NULL;

```

svcinfo中存储了所有“service代理”的信息。 client进程通过binder driver 查找时，就是通过遍历svclist来确定是否有匹配的服务，如果有的话就会返回对应的IBinder。

# service查找逻辑

查找service逻辑对应着client层的getService方法。

```
int svcmgr_handler(struct binder_state *bs,
                   struct binder_transaction_data *txn,
                   struct binder_io *msg,
                   struct binder_io *reply)
{<!-- -->
——————————————省略
    switch(txn-&gt;code) {<!-- -->
    case SVC_MGR_GET_SERVICE:
    case SVC_MGR_CHECK_SERVICE:
        s = bio_get_string16(msg, &amp;len);
        if (s == NULL) {<!-- -->
            return -1;
        }
        handle = do_find_service(s, len, txn-&gt;sender_euid, txn-&gt;sender_pid);
        if (!handle)
            break;
        bio_put_ref(reply, handle);
        return 0;
——————————————省略
        }
——————————————省略
}

```

SVC_MGR_GET_SERVICE与SVC_MGR_CHECK_SERVICE分别对应着 ServiceManager的 GET_SERVICE_TRANSACTION与CHECK_SERVICE_TRANSACTION。 在client调用了ServiceManager的这两个code之后，就会进入到service_manager.c的上面的逻辑中。

>  
 不了解ServiceManager推荐看下前两篇文章：   


其主要逻辑如下：
1. 通过bio_get_string16获取到ipc的字段s（这里的s实际是service的name）1. 通过do_find_service方法获取到handle（即service的句柄）1. 通过bio_put_ref将handle返回
获取handle的逻辑是是do_find_service方法，我们看下这个方法：

### do_find_service

```
uint32_t do_find_service(const uint16_t *s, size_t len, uid_t uid, pid_t spid)
{<!-- -->
    struct svcinfo *si = find_svc(s, len);

    if (!si || !si-&gt;handle) {<!-- -->
        return 0;
    }

    if (!si-&gt;allow_isolated) {<!-- -->
        // If this service doesn't allow access from isolated processes,
        // then check the uid to see if it is isolated.
        uid_t appid = uid % AID_USER;
        if (appid &gt;= AID_ISOLATED_START &amp;&amp; appid &lt;= AID_ISOLATED_END) {<!-- -->
            return 0;
        }
    }

    if (!svc_can_find(s, len, spid, uid)) {<!-- -->
        return 0;
    }

    return si-&gt;handle;
}

```

主要逻辑如下：
1. 通过find_svc找到svcInfo1. 检查service是否可以孤立于进程存在1. 通过svc_can_find检查查询权限1. 返回svcInfo的handle(即句柄)
与存储结构相关的就是find_svc()方法，我们先只看下这个。

### find_svc

```
struct svcinfo *find_svc(const uint16_t *s16, size_t len)
{<!-- -->
    struct svcinfo *si;

    for (si = svclist; si; si = si-&gt;next) {<!-- -->
        if ((len == si-&gt;len) &amp;&amp;
            !memcmp(s16, si-&gt;name, len * sizeof(uint16_t))) {<!-- -->
            return si;
        }
    }
    return NULL;
}

```

如上find_svc()就是拿ipc传过来的strig与svcinfo中的name 比较，如果两者相同，那么就返回这个svcInfo。

这时候再整体看下，其实service_manager中查找service的逻辑其实就是 遍历svclist，用ipc传进来的name 去与svcInfo的name比较，如果name相同就表示找到了需要的svcInfo，返回其handle，即service的句柄。

# service添加逻辑

service添加逻辑对应着client层的addService方法。 我们还是定位到svcmgr_handler中的代码。

```
int svcmgr_handler(struct binder_state *bs,
                   struct binder_transaction_data *txn,
                   struct binder_io *msg,
                   struct binder_io *reply)
{<!-- -->
——————————————省略
    switch(txn-&gt;code) {<!-- -->
——————————————省略
    case SVC_MGR_ADD_SERVICE:
        s = bio_get_string16(msg, &amp;len);
        if (s == NULL) {<!-- -->
            return -1;
        }
        handle = bio_get_ref(msg);
        allow_isolated = bio_get_uint32(msg) ? 1 : 0;
        dumpsys_priority = bio_get_uint32(msg);
        if (do_add_service(bs, s, len, handle, txn-&gt;sender_euid, allow_isolated, dumpsys_priority,
                           txn-&gt;sender_pid))
            return -1;
        break;
——————————————省略
        }
——————————————省略
}

```

SVC_MGR_ADD_SERVICE 对应着ServiceManager中的ADD_SERVICE_TRANSACTION。

>  
 不了解ServiceManager推荐看下前两篇文章：   


这段代码的主要逻辑是：
1. 通过bio_get_xxx等方法，获取到s、handle等ipc的字段.(这里的s实际是service的name)1. 通过do_add_service方法，传入各字段。实现service的添加。
接着我们继续看下do_add_service的具体实现。

### do_add_service

```
int do_add_service(struct binder_state *bs, const uint16_t *s, size_t len, uint32_t handle,
                   uid_t uid, int allow_isolated, uint32_t dumpsys_priority, pid_t spid) {<!-- -->
    struct svcinfo *si;

    if (!handle || (len == 0) || (len &gt; 127))
        return -1;

    if (!svc_can_register(s, len, spid, uid)) {<!-- -->
        ALOGE("add_service('%s',%x) uid=%d - PERMISSION DENIED\n",
             str8(s, len), handle, uid);
        return -1;
    }

    si = find_svc(s, len);
    if (si) {<!-- -->
        if (si-&gt;handle) {<!-- -->
            ALOGE("add_service('%s',%x) uid=%d - ALREADY REGISTERED, OVERRIDE\n",
                 str8(s, len), handle, uid);
            svcinfo_death(bs, si);
        }
        si-&gt;handle = handle;
    } else {<!-- -->
        si = malloc(sizeof(*si) + (len + 1) * sizeof(uint16_t));
        if (!si) {<!-- -->
            ALOGE("add_service('%s',%x) uid=%d - OUT OF MEMORY\n",
                 str8(s, len), handle, uid);
            return -1;
        }
        si-&gt;handle = handle;
        si-&gt;len = len;
        memcpy(si-&gt;name, s, (len + 1) * sizeof(uint16_t));
        si-&gt;name[len] = '\0';
        si-&gt;death.func = (void*) svcinfo_death;
        si-&gt;death.ptr = si;
        si-&gt;allow_isolated = allow_isolated;
        si-&gt;dumpsys_priority = dumpsys_priority;
        si-&gt;next = svclist;
        svclist = si;
    }

    binder_acquire(bs, handle);
    binder_link_to_death(bs, handle, &amp;si-&gt;death);
    return 0;
}

```

主要逻辑如下：
1. 通过svc_can_register检查注册权限1. 通过find_svc查找下服务是否已经存在1. 如果服务已经存在，那么通过svcinfo_death释放原来的服务，然后handle替换成现在service的handle。(即替换句柄)1. 如果服务原本没有存在，那么创建一个svcInfo并且赋值，链接到svclist的头部。1. 调用binder_acquire与binder_link_to_death。（对这两个方法有兴趣的读者可以自行看下）
# 入口函数 service_manager.main()

main()方法是service_manager的入口函数，代码如下：

```
int main(int argc, char** argv)
{<!-- -->
    struct binder_state *bs;
    union selinux_callback cb;
    char *driver;

    if (argc &gt; 1) {<!-- -->
        driver = argv[1];
    } else {<!-- -->
        driver = "/dev/binder";
    }

    bs = binder_open(driver, 128*1024);
    if (!bs) {<!-- -->
#ifdef VENDORSERVICEMANAGER
        ALOGW("failed to open binder driver %s\n", driver);
        while (true) {<!-- -->
            sleep(UINT_MAX);
        }
#else
        ALOGE("failed to open binder driver %s\n", driver);
#endif
        return -1;
    }

    if (binder_become_context_manager(bs)) {<!-- -->
        ALOGE("cannot become context manager (%s)\n", strerror(errno));
        return -1;
    }

    cb.func_audit = audit_callback;
    selinux_set_callback(SELINUX_CB_AUDIT, cb);
    cb.func_log = selinux_log_callback;
    selinux_set_callback(SELINUX_CB_LOG, cb);

#ifdef VENDORSERVICEMANAGER
    sehandle = selinux_android_vendor_service_context_handle();
#else
    sehandle = selinux_android_service_context_handle();
#endif
    selinux_status_open(true);

    if (sehandle == NULL) {<!-- -->
        ALOGE("SELinux: Failed to acquire sehandle. Aborting.\n");
        abort();
    }

    if (getcon(&amp;service_manager_context) != 0) {<!-- -->
        ALOGE("SELinux: Failed to acquire service_manager context. Aborting.\n");
        abort();
    }


    binder_loop(bs, svcmgr_handler);

    return 0;
}

```

主要逻辑如下：
1. 通过binder_open打开binder驱动1. selinux相关逻辑。（有兴趣的读者可以自行了解下）1. 通过binder_become_context_manager，让service_manager成为binder_driver的上下文的管理者。1. 通过binder_loop 进入无限循环，处理client发来的请求。（对于service_manager来说，其他所有进程都是client）
后续再继续深入看下三个逻辑：
1. binder_open()1. binder_become_context_manager()1. binder_loop()
### binder.binder_open()

binder_open这个方法是在binder.c文件中，其文件目录如下： /frameworks/native/cmds/servicemanager/binder.c

```
struct binder_state *binder_open(const char* driver, size_t mapsize)
{<!-- -->
    struct binder_state *bs;
    struct binder_version vers;

    bs = malloc(sizeof(*bs));
    if (!bs) {<!-- -->
        errno = ENOMEM;
        return NULL;
    }

    bs-&gt;fd = open(driver, O_RDWR | O_CLOEXEC);
    if (bs-&gt;fd &lt; 0) {<!-- -->
        fprintf(stderr,"binder: cannot open %s (%s)\n",
                driver, strerror(errno));
        goto fail_open;
    }

    if ((ioctl(bs-&gt;fd, BINDER_VERSION, &amp;vers) == -1) ||
        (vers.protocol_version != BINDER_CURRENT_PROTOCOL_VERSION)) {<!-- -->
        fprintf(stderr,
                "binder: kernel driver version (%d) differs from user space version (%d)\n",
                vers.protocol_version, BINDER_CURRENT_PROTOCOL_VERSION);
        goto fail_open;
    }

    bs-&gt;mapsize = mapsize;
    bs-&gt;mapped = mmap(NULL, mapsize, PROT_READ, MAP_PRIVATE, bs-&gt;fd, 0);
    if (bs-&gt;mapped == MAP_FAILED) {<!-- -->
        fprintf(stderr,"binder: cannot map device (%s)\n",
                strerror(errno));
        goto fail_map;
    }

    return bs;

fail_map:
    close(bs-&gt;fd);
fail_open:
    free(bs);
    return NULL;
}

```

主要逻辑如下：
1. 调用open()尝试打开binder 驱动，如果打不开就返回失败。1. 通过ioctl获取biner 驱动的binder版本，与当前进程的binder版本比较，如果不一样就返回失败。1. 使用mmap()方法来实现内存映射，如果映射直白也就直接返回失败。
### binder.binder_become_context_manager()

binder_become_context_manager这个方法是在binder.c文件中，其文件目录如下： /frameworks/native/cmds/servicemanager/binder.c

```
int binder_become_context_manager(struct binder_state *bs)
{<!-- -->
    return ioctl(bs-&gt;fd, BINDER_SET_CONTEXT_MGR, 0);
}

```

binder_become_context_manager的逻辑： 通过给binder driver来发送BINDER_SET_CONTEXT_MGR的code，将service_manager设置成binder driver的上下文管理者。

>  
 至于binder driver中具体是做了哪些操作，有兴趣的读者可以自行阅读源码了解下。 


### binder.binder_loop()

binder_loop这个方法是在binder.c文件中，其文件目录如下： /frameworks/native/cmds/servicemanager/binder.c

```
void binder_loop(struct binder_state *bs, binder_handler func)
{<!-- -->
    int res;
    struct binder_write_read bwr;
    uint32_t readbuf[32];

    bwr.write_size = 0;
    bwr.write_consumed = 0;
    bwr.write_buffer = 0;

    readbuf[0] = BC_ENTER_LOOPER;
    binder_write(bs, readbuf, sizeof(uint32_t));

    for (;;) {<!-- -->
        bwr.read_size = sizeof(readbuf);
        bwr.read_consumed = 0;
        bwr.read_buffer = (uintptr_t) readbuf;

        res = ioctl(bs-&gt;fd, BINDER_WRITE_READ, &amp;bwr);

        if (res &lt; 0) {<!-- -->
            ALOGE("binder_loop: ioctl failed (%s)\n", strerror(errno));
            break;
        }

        res = binder_parse(bs, 0, (uintptr_t) readbuf, bwr.read_consumed, func);
        if (res == 0) {<!-- -->
            ALOGE("binder_loop: unexpected reply?!\n");
            break;
        }
        if (res &lt; 0) {<!-- -->
            ALOGE("binder_loop: io error %d %s\n", res, strerror(errno));
            break;
        }
    }
}

```

主要逻辑如下：
1. 首先向binder driver发送BC_ENTER_LOOPER，通知binder driver 循环开始。1. 开始进入循环，在循环中发送BINDER_WRITE_READ，向binder driver读取信息。1. 将读取到的信息放到binder_parse方法中解析，然后进入下一个循环。
接着看下解析的方法。

### binder.binder_parse（）

```
int binder_parse(struct binder_state *bs, struct binder_io *bio,
                 uintptr_t ptr, size_t size, binder_handler func)
{<!-- -->
————————————————省略
    switch(cmd) {<!-- -->
        case BR_TRANSACTION: {<!-- -->
            struct binder_transaction_data *txn = (struct binder_transaction_data *) ptr;
            if ((end - ptr) &lt; sizeof(*txn)) {<!-- -->
                ALOGE("parse: txn too small!\n");
                return -1;
            }
            binder_dump_txn(txn);
            if (func) {<!-- -->
                unsigned rdata[256/4];
                struct binder_io msg;
                struct binder_io reply;
                int res;

                bio_init(&amp;reply, rdata, sizeof(rdata), 4);
                bio_init_from_txn(&amp;msg, txn);
                res = func(bs, txn, &amp;msg, &amp;reply);
                if (txn-&gt;flags &amp; TF_ONE_WAY) {<!-- -->
                    binder_free_buffer(bs, txn-&gt;data.ptr.buffer);
                } else {<!-- -->
                    binder_send_reply(bs, &amp;reply, txn-&gt;data.ptr.buffer, res);
                }
            }
            ptr += sizeof(*txn);
            break;
        }
————————————————省略
}

```

binder_parse的逻辑较多，笔者此处主要说一个与service_manager相关的地方。

在cmd为BR_TRANSACTION时，会调用 func方法。 binder_parse中的func方法是binder_loop中的参数 func。 而binder_loop的参数则是svcmgr_handler。

```
int main(int argc, char** argv)
{<!-- -->
————————————省略
    binder_loop(bs, svcmgr_handler);
————————————省略
}

```

svcmgr_handler方法就是，处理的查找service逻辑与添加service逻辑的地方。

```
int svcmgr_handler(struct binder_state *bs,
                   struct binder_transaction_data *txn,
                   struct binder_io *msg,
                   struct binder_io *reply)
{<!-- -->
————————————————————————省略
    switch(txn-&gt;code) {<!-- -->
    case SVC_MGR_GET_SERVICE:
    case SVC_MGR_CHECK_SERVICE:
————————————————————————省略
}

```

# 总结

本文主要讲了service的查找，service的注册 和入口函数main。整理的要点如下：
1. service相关信息 都存储在svcInfo这个结构体中，service_manager维护一个svcInfo的链表 svclist。1. svcInfo中存储的是service对应的handle，即句柄。1. getService：通过name来svclist中查找service，如果找到就返回handle。1. addService：判断该name的service是否存在，如果存在就替换handle，否则就重新创新一个svcInfo放入svclist。1. service_manager在启动后，会无限循环去处理client发来的请求。 getService以及addService等请求都是在这里处理的。
# 继续探索

还有很多其他方面是没有讲到，有兴趣的读者可以自行探索下。
- service的鉴权机制是如何做的？- service的死亡通知是如何实现的？