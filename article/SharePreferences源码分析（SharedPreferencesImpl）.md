#SharePreferences源码分析（SharedPreferencesImpl）
# SharePreferences的基本使用

在Android提供的几种数据存储方式中SharePreference属于轻量级的键值存储方式，以XML文件方式保存数据，通常用来存储一些用户行为开关状态等，一般的存储一些常见的数据类型。

```

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        SharedPreferences sp=getSharedPreferences("data",MODE_PRIVATE);
        SharedPreferences.Editor editor=sp.edit();
        editor.putString("name","xjj");
        editor.commit();

        String s=sp.getString("name","");

    }
}


```

然而，当我们对其实现原理有点兴趣，用"Control+左键"点击看源码时，我们可以发现SharePreferences其实只是一个接口，如下：

```
public interface SharedPreferences {
    Map&lt;String, ?&gt; getAll();
    String getString(String var1, String var2);
    Set&lt;String&gt; getStringSet(String var1, Set&lt;String&gt; var2);
    int getInt(String var1, int var2);
    long getLong(String var1, long var2);
    float getFloat(String var1, float var2);
    boolean getBoolean(String var1, boolean var2);
    boolean contains(String var1);
    SharedPreferences.Editor edit();

    void registerOnSharedPreferenceChangeListener(SharedPreferences.OnSharedPreferenceChangeListener var1);

    void unregisterOnSharedPreferenceChangeListener(SharedPreferences.OnSharedPreferenceChangeListener var1);

    public interface Editor {
        SharedPreferences.Editor putString(String var1, String var2);
        SharedPreferences.Editor putStringSet(String var1, Set&lt;String&gt; var2);
        SharedPreferences.Editor putInt(String var1, int var2);
        SharedPreferences.Editor putLong(String var1, long var2);
        SharedPreferences.Editor putFloat(String var1, float var2);
        SharedPreferences.Editor putBoolean(String var1, boolean var2);
        SharedPreferences.Editor remove(String var1);
        SharedPreferences.Editor clear();
        boolean commit();
        void apply();
    }

    public interface OnSharedPreferenceChangeListener {
        void onSharedPreferenceChanged(SharedPreferences var1, String var2);
    }
}

```

当我们查阅一定的资料，很容易可以发现其实SharedPreferencesImpl才是SharePreferences原理实现的地方。 然而由于SharedPreferencesImpl不是public的类，所以我们无法直接在Android Studio中找到并查看。 但是通过以下方法，我们可以把SharedPreferencesImpl类导入Android Studio。

# 导入SharedPreferencesImpl源码

前往以下路径可以找到SharedPreferencesImpl.java文件，然后直接把文件拖拽到Android Studio中就可以进行查看了。

>  
 android-sdk\sources\android-21\android\app 


# getSharedPreferences()

首先我们要从SharePreferences的获取分析，我们一般使用getSharedPreferences()方法，查看该方法后，代码如下：

```
    @Override
    public SharedPreferences getSharedPreferences(String name, int mode) {
        return mBase.getSharedPreferences(name, mode);
    }

```

```
	public abstract SharedPreferences getSharedPreferences(String name,
            int mode);

```

我们可以发现在Context中的getSharedPreferences（）并没有具体实现，那是因为Context也仅仅是一个接口，它的具体实现是在ContextImpl中。 ContextImpl的导入方法就不多说了，与SharedPreferencesImpl一样，我们直接跳转到ContextImpl中的getSharedPreferences（）方法：

```
    private static ArrayMap&lt;String, ArrayMap&lt;String, SharedPreferencesImpl&gt;&gt; sSharedPrefs;

/*
此处省略很多代码
*/

@Override
    public SharedPreferences getSharedPreferences(String name, int mode) {
        SharedPreferencesImpl sp;
        synchronized (ContextImpl.class) {
            if (sSharedPrefs == null) {
                sSharedPrefs = new ArrayMap&lt;String, ArrayMap&lt;String, SharedPreferencesImpl&gt;&gt;();
            }

            final String packageName = getPackageName();
            ArrayMap&lt;String, SharedPreferencesImpl&gt; packagePrefs = sSharedPrefs.get(packageName);
            if (packagePrefs == null) {
                packagePrefs = new ArrayMap&lt;String, SharedPreferencesImpl&gt;();
                sSharedPrefs.put(packageName, packagePrefs);
            }

            // At least one application in the world actually passes in a null
            // name.  This happened to work because when we generated the file name
            // we would stringify it to "null.xml".  Nice.
            if (mPackageInfo.getApplicationInfo().targetSdkVersion &lt;
                    Build.VERSION_CODES.KITKAT) {
                if (name == null) {
                    name = "null";
                }
            }

            sp = packagePrefs.get(name);
            if (sp == null) {
                File prefsFile = getSharedPrefsFile(name);
                sp = new SharedPreferencesImpl(prefsFile, mode);
                packagePrefs.put(name, sp);
                return sp;
            }
        }
        if ((mode &amp; Context.MODE_MULTI_PROCESS) != 0 ||
            getApplicationInfo().targetSdkVersion &lt; android.os.Build.VERSION_CODES.HONEYCOMB) {
            // If somebody else (some other process) changed the prefs
            // file behind our back, we reload it.  This has been the
            // historical (if undocumented) behavior.
            sp.startReloadIfChangedUnexpectedly();
        }
        return sp;
    }

```

我们可以看到getSharedPreferences通过ContextImpl保证同步操作，所以无论你在一个Context中执行多少次getSharedPreferences（）方法，他们也总是会排序执行，是线程安全的。

另外我们可以看到，SharedPreferencesImpl通过getSharedPrefsFile（）这个方法获取路径，于是对其进行查看如下：

```
    public File getSharedPrefsFile(String name) {
        return makeFilename(getPreferencesDir(), name + ".xml");
    }

```

```
    private File getPreferencesDir() {
        synchronized (mSync) {
            if (mPreferencesDir == null) {
                mPreferencesDir = new File(getDataDirFile(), "shared_prefs");
            }
            return mPreferencesDir;
        }
    }

```

于是我们可以知道我们使用SharePreferences时所存储的路径：

>  
 路径=当前app的data目录下的shared_prefs目录+"/"+SharePreferences的name+“.xml” 


根据如上代码又可以发现ContextImpl中有一个静态的ArrayMap变量sSharedPrefs：

```
    private static ArrayMap&lt;String, ArrayMap&lt;String, SharedPreferencesImpl&gt;&gt; sSharedPrefs;


```

因此无论有多少个ContextImpl对象实例，系统都共享这一个sSharedPrefs的Map，应用启动以后首次使用SharePreference时创建，系统结束时才可能会被垃圾回收器回收，所以如果我们一个App中频繁的使用不同文件名的SharedPreferences很多时这个Map就会很大，也即会占用移动设备宝贵的内存空间，所以说我们应用中应该尽可能少的使用不同文件名的SharedPreferences，减小内存使用。

# SharedPreferencesImpl的实现

知道了SharedPreferences是如何获取的，我们就要开始思考SharedPreferencesImpl是如何实现的了。 主要为以下三个部分：
- SharedPreferencesImpl的构造- 数据的get和put- 数据的commit
## SharedPreferencesImpl的构造：

```
    SharedPreferencesImpl(File file, int mode) {
        mFile = file;
        mBackupFile = makeBackupFile(file);
        mMode = mode;
        mLoaded = false;
        mMap = null;
        startLoadFromDisk();
    }

    private void startLoadFromDisk() {
        synchronized (this) {
            mLoaded = false;
        }
        new Thread("SharedPreferencesImpl-load") {
            public void run() {
                synchronized (SharedPreferencesImpl.this) {
                    loadFromDiskLocked();
                }
            }
        }.start();
    }

    private void loadFromDiskLocked() {
        if (mLoaded) {
            return;
        }
        if (mBackupFile.exists()) {
            mFile.delete();
            mBackupFile.renameTo(mFile);
        }

        // Debugging
        if (mFile.exists() &amp;&amp; !mFile.canRead()) {
            Log.w(TAG, "Attempt to read preferences file " + mFile + " without permission");
        }

        Map map = null;
        StructStat stat = null;
        try {
            stat = Os.stat(mFile.getPath());
            if (mFile.canRead()) {
                BufferedInputStream str = null;
                try {
                    str = new BufferedInputStream(
                            new FileInputStream(mFile), 16*1024);
                    map = XmlUtils.readMapXml(str);
                } catch (XmlPullParserException e) {
                    Log.w(TAG, "getSharedPreferences", e);
                } catch (FileNotFoundException e) {
                    Log.w(TAG, "getSharedPreferences", e);
                } catch (IOException e) {
                    Log.w(TAG, "getSharedPreferences", e);
                } finally {
                    IoUtils.closeQuietly(str);
                }
            }
        } catch (ErrnoException e) {
        }
        mLoaded = true;
        if (map != null) {
            mMap = map;
            mStatTimestamp = stat.st_mtime;
            mStatSize = stat.st_size;
        } else {
            mMap = new HashMap&lt;String, Object&gt;();
        }
        notifyAll();
    }

```

我们可以看到此处**新建了一个线程**，根据我们输入的地址查找是否存在xml的文件，如果可以找到，那就让我这个SharedPreferencesImpl中的mMap等于这个xml中的map。

## 数据的get和put:

### (此处就已getString和putString为例子)

```
    private Map&lt;String, Object&gt; mMap;     // guarded by 'this'
    
    public String getString(String key, String defValue) {
        synchronized (this) {
            awaitLoadedLocked();
            String v = (String)mMap.get(key);
            return v != null ? v : defValue;
        }

```

```
private final Map&lt;String, Object&gt; mModified = Maps.newHashMap();
       
public Editor putString(String key, String value) {
            synchronized (this) {
                mModified.put(key, value);
                return this;
            }
        }

```

这里要说一下，**get和put使用的是对象的同步锁**，就是能保证一个对象在不同线程中进行操作是安全。

## Editor.commit()：

```
        public boolean commit() {
            MemoryCommitResult mcr = commitToMemory();
            SharedPreferencesImpl.this.enqueueDiskWrite(
                mcr, null /* sync write on this thread okay */);
            try {
                mcr.writtenToDiskLatch.await();
            } catch (InterruptedException e) {
                return false;
            }
            notifyListeners(mcr);
            return mcr.writeToDiskResult;
        }

```

进来后，我们发现第一个对象MemoryCommitResult我们就不了解，于是我们看一下它的代码：

```
    // Return value from EditorImpl#commitToMemory()
    private static class MemoryCommitResult {
        public boolean changesMade;  // any keys different?
        public List&lt;String&gt; keysModified;  // may be null
        public Set&lt;OnSharedPreferenceChangeListener&gt; listeners;  // may be null
        public Map&lt;?, ?&gt; mapToWriteToDisk;
        public final CountDownLatch writtenToDiskLatch = new CountDownLatch(1);
        public volatile boolean writeToDiskResult = false;

        public void setDiskWriteResult(boolean result) {
            writeToDiskResult = result;
            writtenToDiskLatch.countDown();
        }
    }

```

我们发现这个类其实很简单，就是存储了一些与EditorImpl有关的值。但是这些值到底是什么呢？我们就要看一下这些值的获取方式了，于是，我们跳转到commitToMemory()方法中：

```
        // Returns true if any changes were made
        private MemoryCommitResult commitToMemory() {
            MemoryCommitResult mcr = new MemoryCommitResult();
            synchronized (SharedPreferencesImpl.this) {
                // We optimistically don't make a deep copy until
                // a memory commit comes in when we're already
                // writing to disk.
                if (mDiskWritesInFlight &gt; 0) {
                    // We can't modify our mMap as a currently
                    // in-flight write owns it.  Clone it before
                    // modifying it.
                    // noinspection unchecked
                    mMap = new HashMap&lt;String, Object&gt;(mMap);
                }
                mcr.mapToWriteToDisk = mMap;
                mDiskWritesInFlight++;

                boolean hasListeners = mListeners.size() &gt; 0;
                if (hasListeners) {
                    mcr.keysModified = new ArrayList&lt;String&gt;();
                    mcr.listeners =
                            new HashSet&lt;OnSharedPreferenceChangeListener&gt;(mListeners.keySet());
                }

                synchronized (this) {
                    if (mClear) {
                        if (!mMap.isEmpty()) {
                            mcr.changesMade = true;
                            mMap.clear();
                        }
                        mClear = false;
                    }

                    for (Map.Entry&lt;String, Object&gt; e : mModified.entrySet()) {
                        String k = e.getKey();
                        Object v = e.getValue();
                        // "this" is the magic value for a removal mutation. In addition,
                        // setting a value to "null" for a given key is specified to be
                        // equivalent to calling remove on that key.
                        if (v == this || v == null) {
                            if (!mMap.containsKey(k)) {
                                continue;
                            }
                            mMap.remove(k);
                        } else {
                            if (mMap.containsKey(k)) {
                                Object existingValue = mMap.get(k);
                                if (existingValue != null &amp;&amp; existingValue.equals(v)) {
                                    continue;
                                }
                            }
                            mMap.put(k, v);
                        }

                        mcr.changesMade = true;
                        if (hasListeners) {
                            mcr.keysModified.add(k);
                        }
                    }

                    mModified.clear();
                }
            }
            return mcr;
        }

```

首先我们要清楚两个Map： **mMap：**是SharedPreferencesImpl构造的时候从xml文件中直接获取到的HashMap。（如果文件中没有，mMap就为一个没有数据的HashMap） **mModified：**是Editor中临时的用于存放提交数据的HashMap。

我们可以看到这里的逻辑首先让**mcr.mapToWriteToDisk=mMap**。 然后我们遍历mModified中的对象（包括key与value两个值），有**三种情况**：

1、如果value为null，并且mMap中包含这个对象，那么就remove。

2、如果value不为null，并且mMap中包含这个对象，并且这个对象的value与原来文件中的value相同。直接continue，跳过。

3、 如果value不为null，并且mMap中包含这个对象，并且这个对象的value与原来文件中的value **不相同**。 或者，如果value不为null，并且mMap中**不包含**这个对象。 这个时候，就需要把这个值加入到mMap中，也就是加入到mcr.mapToWriteToDisk中。

另外，这里我们可以看到commit（）使用的锁和SharedPreferencesImpl的构造是同一把，如下：

```
private void startLoadFromDisk() {
        synchronized (this) {
            mLoaded = false;
        }
        new Thread("SharedPreferencesImpl-load") {
            public void run() {
                synchronized (SharedPreferencesImpl.this) {
                    loadFromDiskLocked();
                }
            }
        }.start();
    }

```

```
 private MemoryCommitResult commitToMemory() {
            MemoryCommitResult mcr = new MemoryCommitResult();
            synchronized (SharedPreferencesImpl.this) {

```

我们再回忆一下： get的内容来自mMap，而mMap是构造的时候从文件读取的。 put的内容只有在commit之后，改变mMap的值，并且写入文件。

然后我们就可以开始后面写入逻辑的分析了，进入enqueueDiskWrite()这个函数：

```
private void enqueueDiskWrite(final MemoryCommitResult mcr,
                                  final Runnable postWriteRunnable) {
        final Runnable writeToDiskRunnable = new Runnable() {
                public void run() {
                    synchronized (mWritingToDiskLock) {
                        writeToFile(mcr);
                    }
                    synchronized (SharedPreferencesImpl.this) {
                        mDiskWritesInFlight--;
                    }
                    if (postWriteRunnable != null) {
                        postWriteRunnable.run();
                    }
                }
            };

        final boolean isFromSyncCommit = (postWriteRunnable == null);

        // Typical #commit() path with fewer allocations, doing a write on
        // the current thread.
        if (isFromSyncCommit) {
            boolean wasEmpty = false;
            synchronized (SharedPreferencesImpl.this) {
                wasEmpty = mDiskWritesInFlight == 1;
            }
            if (wasEmpty) {
                writeToDiskRunnable.run();
                return;
            }
        }

        QueuedWork.singleThreadExecutor().execute(writeToDiskRunnable);
    }

```

分析一下，很容易知道主要逻辑就在writeToFile()这个函数中：

```
    // Note: must hold mWritingToDiskLock
    private void writeToFile(MemoryCommitResult mcr) {
        // Rename the current file so it may be used as a backup during the next read
        if (mFile.exists()) {
            if (!mcr.changesMade) {
                // If the file already exists, but no changes were
                // made to the underlying map, it's wasteful to
                // re-write the file.  Return as if we wrote it
                // out.
                mcr.setDiskWriteResult(true);
                return;
            }
            if (!mBackupFile.exists()) {
                if (!mFile.renameTo(mBackupFile)) {
                    Log.e(TAG, "Couldn't rename file " + mFile
                          + " to backup file " + mBackupFile);
                    mcr.setDiskWriteResult(false);
                    return;
                }
            } else {
                mFile.delete();
            }
        }

        // Attempt to write the file, delete the backup and return true as atomically as
        // possible.  If any exception occurs, delete the new file; next time we will restore
        // from the backup.
        try {
            FileOutputStream str = createFileOutputStream(mFile);
            if (str == null) {
                mcr.setDiskWriteResult(false);
                return;
            }
            XmlUtils.writeMapXml(mcr.mapToWriteToDisk, str);
            FileUtils.sync(str);
            str.close();
            ContextImpl.setFilePermissionsFromMode(mFile.getPath(), mMode, 0);
            try {
                final StructStat stat = Os.stat(mFile.getPath());
                synchronized (this) {
                    mStatTimestamp = stat.st_mtime;
                    mStatSize = stat.st_size;
                }
            } catch (ErrnoException e) {
                // Do nothing
            }
            // Writing was successful, delete the backup file if there is one.
            mBackupFile.delete();
            mcr.setDiskWriteResult(true);
            return;
        } catch (XmlPullParserException e) {
            Log.w(TAG, "writeToFile: Got exception:", e);
        } catch (IOException e) {
            Log.w(TAG, "writeToFile: Got exception:", e);
        }
        // Clean up an unsuccessfully written file
        if (mFile.exists()) {
            if (!mFile.delete()) {
                Log.e(TAG, "Couldn't clean up partially-written file " + mFile);
            }
        }
        mcr.setDiskWriteResult(false);
    }

```

然后就很简单了，首先先查看原来的地址有没有文件存在，如果已经有了，那么就删除。然后把mcr.mapToWriteToDisk这个HashMap对象转化为xml格式存储到文件放到改路径下。

# 总结

1、SharePreferences的xml文件存储路径=当前app的data目录下的shared_prefs目录+SharePreferences的name+“.xml”

2、SharedPreferencesImpl构造中主要是创建mMap对象，会创建一个线程去文件中查找是否存在该xml，如果存在就把xml转化为HashMap，让mMap等于它。如果不存在，就让mMap等于一个空的HashMap。

3、 SharedPreferencesImpl的构造，和commit中使用的是都是SharedPreferencesImpl.class的类的锁，说明这个类创建的所有的对象，同一时间也只能有一个对象在初始化或者commit。

get的内容来自mMap，而mMap是构造的时候从文件读取的。 put的内容只有在commit之后，改变mMap的值，并且写入文件。 get和put都是使用的对象锁。

4、Editor中创建了一个临时的HashMap用于存放要提交的数据。

5、commit中会先获取到mMap对象，然后遍历Editor中临时的用于存放数据的HashMap，发现有改变的数据，就放入mMap对象。同时也会删除value为空的数据。 6、最后writeToFile（）中先会查看原路径是否有文件存在，如果存在就删除。然后把改变后的mMap对象转化为xml格式写入该路径下的文件。

# 拓展

有读者提出想了解一下apply（）与commit（）的区别，由于此篇文章已经发布，就不加长篇幅了。 两者原理文章地址如下： 
