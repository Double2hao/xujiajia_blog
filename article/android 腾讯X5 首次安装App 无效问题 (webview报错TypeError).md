#android 腾讯X5 首次安装App 无效问题 (webview报错TypeError)
# 问题

腾讯X5 webview在两种情况下使用，在首次启动会有问题： 1、应用启动后马上调用webview 2、新启一个进程使用webview，并且在新进程中初始化X5

>  
 此问题只会在首次安装的时候出现，第二次启动App的时候就不会有问题了。 


报错如下：

```
 TypeError: Cannot read property 'getExtension' of null

```

# 原因

首次安装本地没有X5内核可用，如果在X5初始化结束之前调用webview，那么默认会使用系统内核。

初始化demo源码如下：

```
public class APPAplication extends Application {<!-- -->

	@Override
	public void onCreate() {<!-- -->
		// TODO Auto-generated method stub
		super.onCreate();
		//搜集本地tbs内核信息并上报服务器，服务器返回结果决定使用哪个内核。

		QbSdk.PreInitCallback cb = new QbSdk.PreInitCallback() {<!-- -->
			
			@Override
			public void onViewInitFinished(boolean arg0) {<!-- -->
				// TODO Auto-generated method stub
				//x5內核初始化完成的回调，为true表示x5内核加载成功，否则表示x5内核加载失败，会自动切换到系统内核。
				Log.d("app", " onViewInitFinished is " + arg0);
			}
			
			@Override
			public void onCoreInitFinished() {<!-- -->
				// TODO Auto-generated method stub
			}
		};
		//x5内核初始化接口
		QbSdk.initX5Environment(getApplicationContext(),  cb);
	}

}

```

# 解决方式

1、使用全局参数来判断X5是否初始化结束，如果没有初始化结束，那么webview就不加载。（可以报错，可以通过时延加载等） 2、可以使用EventBus在可能出现此问题的地方做处理。如果X5没有初始化完毕就不加载webview，等待X5加载完毕后通过EventBus的事件来加载。

# 注意点

对于高版本的机型如果没有一定要用X5的需求，可以做版本判断，直接使用系统内核。 避免首次安装做无效的时延或者等待，降低用户体验。
