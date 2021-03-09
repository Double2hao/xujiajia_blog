#rxlifecycle bindUntilEvent源码解析
# 概述

rxjava使用时有个常见的问题： 我们经常会在onNext中执行一些ui操作，当执行完耗时操作触发onNext的时候，Activity有可能已经destory了，我们期望是就不去执行onNext中的内容了。

这种操作使用rxlifecycle来实现非常容易，只需一行代码即可：

```
observable.compose(this.&lt;String&gt;bindUntilEvent(ActivityEvent.DESTROY))

```

依赖rxjava以及rxlifecycle代码如下：

```
    implementation "io.reactivex.rxjava2:rxjava:2.2.13"
    implementation 'com.trello.rxlifecycle3:rxlifecycle-components:3.1.0'

```

# demo

使用需要两步：
1. Activity继承RxAppCompatActivity1. 在compose中调用bindUntilEvent。即传入你需要解绑的生命周期。
```
public class MainActivity extends RxAppCompatActivity {<!-- -->
  private static final String TAG = "MainActivity";

  private TextView tvMain;

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    initViews();
    getDataByRx();
  }

  private void initViews() {<!-- -->
    tvMain = findViewById(R.id.tv_main);
  }

  private void getDataByRx() {<!-- -->
    Observable.create(new ObservableOnSubscribe&lt;String&gt;() {<!-- -->
      @Override
      public void subscribe(@NonNull ObservableEmitter&lt;String&gt; e) throws Exception {<!-- -->
        //使用时延来模拟网络操作
        Thread.sleep(3000);
        String s = "用来模拟网络操作后获取到的内容";
        e.onNext(s);
        e.onComplete();
      }
    }).subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .compose(this.&lt;String&gt;bindUntilEvent(ActivityEvent.DESTROY))
        .subscribe(new Observer&lt;String&gt;() {<!-- -->
          @Override public void onSubscribe(Disposable d) {<!-- -->

          }

          @Override public void onNext(String s) {<!-- -->
            if (tvMain != null) {<!-- -->
              tvMain.setText(s);
            }
          }

          @Override public void onError(Throwable e) {<!-- -->
            Log.e(TAG, String.valueOf(e.getMessage()));
          }

          @Override public void onComplete() {<!-- -->
            Log.d(TAG, "onComplete");
          }
        });
  }
}

```

# bindUntilEvent()

直接看bindUntilEvent源码：

```
RxAppCompatActivity

  public final &lt;T&gt; LifecycleTransformer&lt;T&gt; bindUntilEvent(@NonNull ActivityEvent event) {<!-- -->
    return RxLifecycle.bindUntilEvent(this.lifecycleSubject, event);
  }

```

```
RxLifecycle

    public static &lt;T, R&gt; LifecycleTransformer&lt;T&gt; bindUntilEvent(@Nonnull final Observable&lt;R&gt; lifecycle,
                                                                @Nonnull final R event) {<!-- -->
        checkNotNull(lifecycle, "lifecycle == null");
        checkNotNull(event, "event == null");
        return bind(takeUntilEvent(lifecycle, event));
    }
    
    private static &lt;R&gt; Observable&lt;R&gt; takeUntilEvent(final Observable&lt;R&gt; lifecycle, final R event) {<!-- -->
        return lifecycle.filter(new Predicate&lt;R&gt;() {<!-- -->
            @Override
            public boolean test(R lifecycleEvent) throws Exception {<!-- -->
                return lifecycleEvent.equals(event);
            }
        });
    }

```

这里我们可以看到event最终是传递到了 test()这个方法中，于是我们将断点打到test这个方法中，来看下最终是哪里调用刚到了这个方法。

# 最终调用

```
ObservableFilter.FilterObserver

        @Override
        public void onNext(T t) {<!-- -->
            if (sourceMode == NONE) {<!-- -->
                boolean b;
                try {<!-- -->
                    b = filter.test(t);
                } catch (Throwable e) {<!-- -->
                    fail(e);
                    return;
                }
                if (b) {<!-- -->
                    downstream.onNext(t);
                }
            } else {<!-- -->
                downstream.onNext(null);
            }
        }

```

在这里，一旦test返回true。（在例子中，即是Activity到了destory中） 最终会调用到这行代码：

```
downstream.onNext(t);

```

通过断点进入到如下代码：

```
ObservableTakeUntil.OtherObserver

            @Override
            public void onNext(U t) {<!-- -->
                DisposableHelper.dispose(this);
                otherComplete();
            }

```

```
ObservableTakeUntil
        void otherComplete() {<!-- -->
            DisposableHelper.dispose(upstream);
            HalfSerializer.onComplete(downstream, this, error);
        }

```

到这里，其实结果已经出来了。 bindUntilEvent中传入了event后，在该event到达后，会执行两个操作：
1. dispose事件1. 执行onComplete
# 源码阅读收获
1. 可以通过dispose直接根据自己需要取消掉事件。1. 使用rxlifecycle 的bindUntilEvent() 后，最终还是会调用到onCompete方法，因此onComplete中要注意防null 和 防止内存泄漏。