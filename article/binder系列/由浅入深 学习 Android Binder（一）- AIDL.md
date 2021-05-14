#由浅入深 学习 Android Binder（一）- AIDL
>  
 Android Binder系列文章：            


# 概述

aidl是常用的android IPC方式，本文将根据一个demo来解析下AIDL的原理。

为了便于读者理解，本文不会探究Binder的实现细节，可以认为Binder在此文的分析中被看做是一个“黑盒”。

有一定经验的读者可以直接到文末看总结，最终流程图如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911915923.png " alt="在这里插入图片描述">

# Demo

demo实现内容： MainActivity通过AIDL实现IPC来调用另一个进程中RemoteTestService的一个方法。 主要有以下几个文件：
- MainActivity.java 自定义ServiceConnection，然后将ServiceConnection传入bindService，获取到IBinder后实现远程调用。- RemoteTestService.java 在ITestServer.Stub中实现需要远程调用的方法testFunction()，在onBind中返回。- IServer.aidl 定义一个aidl文件和需要远程调用的方法- AndroidManifest.xml 在此设置RemoteTestService在remote进程。
### MainActivity.java

```
public class MainActivity extends AppCompatActivity {<!-- -->

  private IServer iServer;
  private ServiceConnection serviceConnection = new ServiceConnection() {<!-- -->
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {<!-- -->
      iServer = IServer.Stub.asInterface(service);
      System.out.println("onServiceConnected");

      try {<!-- -->
        int a = iServer.testFunction("test string");
        Log.i("test", "after testFunction a:" + a);
      } catch (Exception e) {<!-- -->
        e.printStackTrace();
      }
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {<!-- -->
      Log.i("test", "onServiceDisconnected");
    }
  };

  @Override
  protected void onCreate(Bundle savedInstanceState) {<!-- -->
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Intent intent = new Intent(MainActivity.this, RemoteTestService.class);
    bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE);
  }
}

```

### RemoteTestService.java

```
public class RemoteTestService extends Service {<!-- -->

  private IServer.Stub serverBinder = new IServer.Stub() {<!-- -->
    @Override public int testFunction(String s) throws RemoteException {<!-- -->
      Log.i("test","testFunction s= " + s);
      return 0;
    }
  };

  @Nullable @Override
  public IBinder onBind(Intent intent) {<!-- -->
    return serverBinder;
  }
}

```

### IServer.aidl

```
interface IServer {<!-- -->
    int testFunction(String s);
}

```

### AndroidManifest.xml

```
&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.bindservicetest"&gt;

  &lt;application
      android:allowBackup="true"
      android:icon="@mipmap/ic_launcher"
      android:label="@string/app_name"
      android:roundIcon="@mipmap/ic_launcher_round"
      android:supportsRtl="true"
      android:theme="@style/AppTheme"&gt;
    &lt;activity android:name=".MainActivity"&gt;
      &lt;intent-filter&gt;
        &lt;action android:name="android.intent.action.MAIN" /&gt;

        &lt;category android:name="android.intent.category.LAUNCHER" /&gt;
      &lt;/intent-filter&gt;
    &lt;/activity&gt;

    &lt;service
        android:name=".RemoteTestService"
        android:process=":remote" /&gt;
  &lt;/application&gt;


&lt;/manifest&gt;

```

# aidl自动生成的文件

定义完aidl文件，编译会自动生成一个java接口文件。 这个接口文件在build目录下，具体路径如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911913541.png " width="50%" height="50%">

打开文件，我们就可以看到aidl自动生成的代码。

```
public interface IServer extends android.os.IInterface
{<!-- -->
  /** Default implementation for IServer. */
  public static class Default implements com.example.bindservicetest.IServer
  {<!-- -->
    @Override public int testFunction(java.lang.String s) throws android.os.RemoteException
    {<!-- -->
      return 0;
    }
    @Override
    public android.os.IBinder asBinder() {<!-- -->
      return null;
    }
  }
  /** Local-side IPC implementation stub class. */
  public static abstract class Stub extends android.os.Binder implements com.example.bindservicetest.IServer
  {<!-- -->
    private static final java.lang.String DESCRIPTOR = "com.example.bindservicetest.IServer";
    /** Construct the stub at attach it to the interface. */
    public Stub()
    {<!-- -->
      this.attachInterface(this, DESCRIPTOR);
    }
    /**
     * Cast an IBinder object into an com.example.bindservicetest.IServer interface,
     * generating a proxy if needed.
     */
    public static com.example.bindservicetest.IServer asInterface(android.os.IBinder obj)
    {<!-- -->
      if ((obj==null)) {<!-- -->
        return null;
      }
      android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
      if (((iin!=null)&amp;&amp;(iin instanceof com.example.bindservicetest.IServer))) {<!-- -->
        return ((com.example.bindservicetest.IServer)iin);
      }
      return new com.example.bindservicetest.IServer.Stub.Proxy(obj);
    }
    @Override public android.os.IBinder asBinder()
    {<!-- -->
      return this;
    }
    @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
    {<!-- -->
      java.lang.String descriptor = DESCRIPTOR;
      switch (code)
      {<!-- -->
        case INTERFACE_TRANSACTION:
        {<!-- -->
          reply.writeString(descriptor);
          return true;
        }
        case TRANSACTION_testFunction:
        {<!-- -->
          data.enforceInterface(descriptor);
          java.lang.String _arg0;
          _arg0 = data.readString();
          int _result = this.testFunction(_arg0);
          reply.writeNoException();
          reply.writeInt(_result);
          return true;
        }
        default:
        {<!-- -->
          return super.onTransact(code, data, reply, flags);
        }
      }
    }
    private static class Proxy implements com.example.bindservicetest.IServer
    {<!-- -->
      private android.os.IBinder mRemote;
      Proxy(android.os.IBinder remote)
      {<!-- -->
        mRemote = remote;
      }
      @Override public android.os.IBinder asBinder()
      {<!-- -->
        return mRemote;
      }
      public java.lang.String getInterfaceDescriptor()
      {<!-- -->
        return DESCRIPTOR;
      }
      @Override public int testFunction(java.lang.String s) throws android.os.RemoteException
      {<!-- -->
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        int _result;
        try {<!-- -->
          _data.writeInterfaceToken(DESCRIPTOR);
          _data.writeString(s);
          boolean _status = mRemote.transact(Stub.TRANSACTION_testFunction, _data, _reply, 0);
          if (!_status &amp;&amp; getDefaultImpl() != null) {<!-- -->
            return getDefaultImpl().testFunction(s);
          }
          _reply.readException();
          _result = _reply.readInt();
        }
        finally {<!-- -->
          _reply.recycle();
          _data.recycle();
        }
        return _result;
      }
      public static com.example.bindservicetest.IServer sDefaultImpl;
    }
    static final int TRANSACTION_testFunction = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    public static boolean setDefaultImpl(com.example.bindservicetest.IServer impl) {<!-- -->
      if (Stub.Proxy.sDefaultImpl == null &amp;&amp; impl != null) {<!-- -->
        Stub.Proxy.sDefaultImpl = impl;
        return true;
      }
      return false;
    }
    public static com.example.bindservicetest.IServer getDefaultImpl() {<!-- -->
      return Stub.Proxy.sDefaultImpl;
    }
  }
  public int testFunction(java.lang.String s) throws android.os.RemoteException;
}

```

# 根据源码的简洁版来解析

>  
 aidl文件本身并不会参与代码的运行，编译后自动生成的文件才是关键，因此这个文件由开发者来手动实现也是完全可以的。 


自动生成的接口文件,有较多代码在demo的IPC中并没有使用。 因此笔者打算用源码解析就用自己写的接口文件，也是减少给读者的混淆项。 笔者自己实现的 接口ITestServer.java，源码如下：

```
public interface ITestServer extends android.os.IInterface {<!-- -->

  public int testFunction(java.lang.String s) throws android.os.RemoteException;

  public static abstract class Stub extends android.os.Binder
      implements com.example.bindservicetest.ITestServer {<!-- -->
    //binder唯一标识
    private static final java.lang.String DESCRIPTOR = "com.example.bindservicetest.ITestServer";
    //testFunction的调用id
    static final int TRANSACTION_testFunction = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);

    public Stub() {<!-- -->
      //将interface提供出去，这样当统一进程其他位置执行IBinder.queryLocalInterface的时候就可以获取到这个Binder
      this.attachInterface(this, DESCRIPTOR);
    }

    public static com.example.bindservicetest.ITestServer asInterface(android.os.IBinder obj) {<!-- -->
      if ((obj == null)) {<!-- -->
        return null;
      }
      android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);//查找当前进程是否有这个binder
      if (((iin != null) &amp;&amp; (iin instanceof com.example.bindservicetest.IServer))) {<!-- -->
        return ((com.example.bindservicetest.ITestServer) iin);
      }
      return new com.example.bindservicetest.ITestServer.Stub.Proxy(obj);
    }

    @Override public android.os.IBinder asBinder() {<!-- -->
      return this;//继承自IInterface，返回当前Stub的binder对象
    }

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

    private static class Proxy implements com.example.bindservicetest.ITestServer {<!-- -->
      private final android.os.IBinder mRemote;

      Proxy(android.os.IBinder remote) {<!-- -->
        mRemote = remote;
      }

      @Override public android.os.IBinder asBinder() {<!-- -->
        return mRemote;//继承自IInterface，返回当前Proxy的binder对象
      }

      @Override public int testFunction(java.lang.String s) throws android.os.RemoteException {<!-- -->
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        int _result;
        try {<!-- -->
          _data.writeInterfaceToken(DESCRIPTOR);//需要传递DESCRIPTOR，到另一个进程可以找到对应的Binder
          _data.writeString(s);//写入参数
          boolean _status =
              mRemote.transact(Stub.TRANSACTION_testFunction, _data, _reply, 0);//通过transact来IPC调用
          _reply.readException();
          _result = _reply.readInt();//读取运行结果
        } finally {<!-- -->
          _reply.recycle();
          _data.recycle();
        }
        return _result;
      }
    }
  }
}

```

# ITestServer 源码解析

通过android studio，可以看到该类的结构如下。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911914522.png " width="50%" height="50%">

接下来我们就主要分析三个类：
- ITestServer Stub和Proxy都implement了它。 java上层可以使用这个接口来保留Binder的对象。在使用aidl的时候，完全可以只使用这个对象而不用关心这个对象是Stub还是Proxy。- ITestServer.Stub 当前进程的Binder对象。 在onTransact方法中实现了对IPC方法的逻辑，Proxy调用的时候会触发。- ITestServer.Stub.Proxy 当前进程通过Proxy实现了对远端方法的调用。 当前进程调用远端进程方法，实际上是调用了远端Binder的transact方法，最终执行到Stub中的onTransact。
### ITestServer

```
public interface ITestServer extends android.os.IInterface {<!-- -->

  public int testFunction(java.lang.String s) throws android.os.RemoteException;

----------------Stub的代码先不看
}

```

```
/**
 * Base class for Binder interfaces.  When defining a new interface,
 * you must derive it from IInterface.
 */
public interface IInterface
{<!-- -->
    /**
     * Retrieve the Binder object associated with this interface.
     * You must use this instead of a plain cast, so that proxy objects
     * can return the correct result.
     */
    public IBinder asBinder();
}

```

要点如下：
- ITestServer本身只包括“需要IPC的方法”，此处即int testFunction(String)- 这个类继承了IInterface，需要实现asBinder接口。 asBinder()会返回正确的Binder对象。如果是同进程调用就会直接返回当前进程的Binder，如果是IPC，就会返回远程调用的进程的Binder。 对应源码就是，Stub的asBinder()返回的是this，而Proxy返回的是mRemote.
# ITestServer.Stub

### DESCRIPTOR

```
//binder唯一标识
    private static final java.lang.String DESCRIPTOR = "com.example.bindservicetest.ITestServer";

```

是Binder的唯一标识。 不同的进程之间，通过序列化传递DESCRIPTOR来找到对应的Binder。 相同进程，也需要通过DESCRIPTOR才找到对应的Binder。

### TRANSACTION_testFunction

```
//testFunction的调用id
    static final int TRANSACTION_testFunction = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);

```

调用IPC方法的唯一id。 Stub.onTransact()中会通过TRANSACTION_testFunction来对应 执行testFunction()方法的逻辑。 Proxy调用transact的时候，也是通过传递TRANSACTION_testFunction，来标识自己想要执行的逻辑。

### Stub初始化

```
    public Stub() {<!-- -->
      //将interface提供出去，这样当统一进程其他位置执行IBinder.queryLocalInterface的时候就可以获取到这个Binder
      this.attachInterface(this, DESCRIPTOR);
    }

```

```
    /**
     * Convenience method for associating a specific interface with the Binder.
     * After calling, queryLocalInterface() will be implemented for you
     * to return the given owner IInterface when the corresponding
     * descriptor is requested.
     */
    public void attachInterface(@Nullable IInterface owner, @Nullable String descriptor) {<!-- -->
        mOwner = owner;
        mDescriptor = descriptor;
    }
    
        /**
     * Use information supplied to attachInterface() to return the
     * associated IInterface if it matches the requested
     * descriptor.
     */
    public @Nullable IInterface queryLocalInterface(@NonNull String descriptor) {<!-- -->
        if (mDescriptor != null &amp;&amp; mDescriptor.equals(descriptor)) {<!-- -->
            return mOwner;
        }
        return null;
    }

```

初始化时，调用attachInterface()。 attachInterface()之后，相同进程调用queryLocalInterface()的时候就能找到这个Binder。 在下面asInterface()中，就用到了queryLocalInterface()。

### asInterface()

```
    public static com.example.bindservicetest.ITestServer asInterface(android.os.IBinder obj) {<!-- -->
      if ((obj == null)) {<!-- -->
        return null;
      }
      android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);//查找当前进程是否有这个binder
      if (((iin != null) &amp;&amp; (iin instanceof com.example.bindservicetest.IServer))) {<!-- -->
        return ((com.example.bindservicetest.ITestServer) iin);
      }
      return new com.example.bindservicetest.ITestServer.Stub.Proxy(obj);
    }

```

用于将服务端的IBinder对象转化成ITestServer对象。 如果是server和client进程相同，那么会直接通过queryLocalInterface(DESCRIPTOR)找到Binder返回，这里返回的其实就是Stub对象。 如果server和client在不同进程，那么会返回一个封装后的Proxy对象。

### asBinder()

```
    @Override public android.os.IBinder asBinder() {<!-- -->
      return this;//继承自IInterface，返回当前Stub的binder对象
    }

```

返回Stub自己。 在Proxy中返回的是远端进程的Binder。

### onTransact()

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

实现了被IPC调用后的逻辑。 一个方法的调用在onTransact中一般有以下几个逻辑：
1. 从data中读出参数1. 将读出的参数带入server的方法中，得到执行结果。1. 将执行结果写入reply
>  
 参数和结果为什么读写都需要通过Parcel？ 进程间通信由于资源不共享，因此无法直接传递对象，只能通过序列化在不同的空间拷贝两份相同的资源。 而Parcel就是序列化的一种方案。 


# ITestServer.Stub.Proxy

```
    private static class Proxy implements com.example.bindservicetest.ITestServer {<!-- -->
      private final android.os.IBinder mRemote;

      Proxy(android.os.IBinder remote) {<!-- -->
        mRemote = remote;
      }

      @Override public android.os.IBinder asBinder() {<!-- -->
        return mRemote;//继承自IInterface，返回当前Proxy的binder对象
      }

      @Override public int testFunction(java.lang.String s) throws android.os.RemoteException {<!-- -->
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        int _result;
        try {<!-- -->
          _data.writeInterfaceToken(DESCRIPTOR);//需要传递DESCRIPTOR，到另一个进程可以找到对应的Binder
          _data.writeString(s);//写入参数
          boolean _status =
              mRemote.transact(Stub.TRANSACTION_testFunction, _data, _reply, 0);//通过transact来IPC调用
          _reply.readException();
          _result = _reply.readInt();//读取运行结果
        } finally {<!-- -->
          _reply.recycle();
          _data.recycle();
        }
        return _result;
      }
    }

```

### Proxy初始化

```
      private final android.os.IBinder mRemote;

      Proxy(android.os.IBinder remote) {<!-- -->
        mRemote = remote;
      }

```

Proxy在初始化的时候会引用server的IBinder。

### asBinder()

```
      @Override public android.os.IBinder asBinder() {<!-- -->
        return mRemote;//继承自IInterface，返回当前Proxy的binder对象
      }

```

Proxy在asBinder的时候会返回server的IBinder。 Stub在这个方法中会返回this。

### testFunction()

```
      @Override public int testFunction(java.lang.String s) throws android.os.RemoteException {<!-- -->
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        int _result;
        try {<!-- -->
          _data.writeInterfaceToken(DESCRIPTOR);//需要传递DESCRIPTOR，到另一个进程可以找到对应的Binder
          _data.writeString(s);//写入参数
          boolean _status =
              mRemote.transact(Stub.TRANSACTION_testFunction, _data, _reply, 0);//通过transact来IPC调用
          _reply.readException();
          _result = _reply.readInt();//读取运行结果
        } finally {<!-- -->
          _reply.recycle();
          _data.recycle();
        }
        return _result;
      }

```

Proxy调用testFunction，实际上是通过调用mRemote.transact()来触发远端Stub的onTransact()。 一般调用的步骤如下：
1. 创建参数与返回值的Parcel对象，将参数写入Parcel。1. 调用mRemote.transact()，返回值会写入到Parcel对象中。1. 从Parcel对象中读出返回值并return。
# 总结

根据上文的分析，aidl自动生成的文件的源码大概可以总结成下图：

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209911915923.png " alt="在这里插入图片描述">

基本步骤如下：
1. Client通过ServiceConnection获取到Server的Binder，并且封装成一个Proxy。1. 通过Proxy来同步调用IPC方法。同时通过Parcel将参数传给Binder，最终触发Binder的transact方法。1. Binder的transact方法最终会触发到Server上Stub的onTransact方法。1. Server上Stub的onTransact方法中，会先从Parcel中解析中参数，然后将参数带入真正的方法中执行，然后将结果写入Parcel后传回。1. Client的Ipc方法中，执行Binder的transact时，是阻塞等待的。一直到Server逻辑执行结束后才会继续执行。1. 当Server返回结果后，Client从Parcel中取出返回值，于是实现了一次IPC调用。
# 继续探索

读完本文，细心的读者会发现本文还留有两个较大的问题完全没有提及：
1. ServiceConnection是如何获取到Binder的1. 通过ServiceConnection获取到的Binder是什么，或者说java层的Binder是什么
这两点笔者后面的文章会继续探索，有兴趣的读者可以自行探索或者欢迎持续关注。