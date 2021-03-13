#迟到的Volley源码解析
#前言 笔者其实之前其实早就想对Volley源码解析，但是由于笔者基础不好，理解源码有一定困难，加上不够重视，于是便不了了之了。 在近期笔试面试的经历中，发现网络实践这一块实在重要，于是打算亡羊补牢。也是祭奠下失去的腾讯实习offer(三面+加面，挂在加面)，我们无法责怪运气，能做的只是提高自我。

>  
 参考文章： 
   


#简单使用 关于Volley的简单使用主要以下三步： 1、获取RequestQueue对象，如下

```
RequestQueue mQueue = Volley.newRequestQueue(context);  

```

2、定义Request，需要传入URL，服务器响应成功的回调和服务器响应失败的回调等。具体的有StringRequest，JsonRequest等，如下：

```
StringRequest stringRequest = new StringRequest("http://www.baidu.com",  
                        new Response.Listener&lt;String&gt;() {  
                            @Override  
                            public void onResponse(String response) {  
                                Log.d("TAG", response);  
                            }  
                        }, new Response.ErrorListener() {  
                            @Override  
                            public void onErrorResponse(VolleyError error) {  
                                Log.e("TAG", error.getMessage(), error);  
                            }  
                        });  

```

3、将Request对象添加到RequestQueue里面，如下：

```
mQueue.add(stringRequest); 

```

在此就不多做累述，如果是初学Volley的读者，推荐看下郭霖前辈的博客： 

#初始化：newRequestQueue（） 每次使用Volley第一步我们就是定义一个RequestQueue 对象，如下：

```
RequestQueue mQueue = Volley.newRequestQueue(context);  

```

我们看一下这个方法里面做了什么：

```
 public static RequestQueue newRequestQueue(Context context) {
        return newRequestQueue(context, null);
    }

```

```
 public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
        File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);

        String userAgent = "volley/0";
        try {
            String packageName = context.getPackageName();
            PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
            userAgent = packageName + "/" + info.versionCode;
        } catch (NameNotFoundException e) {
        }

        if (stack == null) {
            if (Build.VERSION.SDK_INT &gt;= 9) {
                stack = new HurlStack();
            } else {
                // Prior to Gingerbread, HttpUrlConnection was unreliable.
                // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
            }
        }

        Network network = new BasicNetwork(stack);

        RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
        queue.start();

        return queue;
    }

```

首先了解下什么是HttpStack。 **HttpStack：**处理 Http 请求，返回请求结果。目前 Volley 中有基于 HttpURLConnection 的HurlStack和 基于 Apache HttpClient 的HttpClientStack。 在源码中我们可以看到，如果手机系统版本号是大于9的，则创建一个HurlStack的实例，否则就创建一个HttpClientStack的实例。而HurlStack的内部就是使用HttpURLConnection进行网络通讯的，HttpClientStack的内部则是使用HttpClient进行网络通讯的。

然后，通过HttpStack生成BasicNetwork对象，接着把BasicNetwork对象传递到RequestQueue这个对象中。 最后调用了start（）方法，代码如下：

```
public void start() {
        stop();  // Make sure any currently running dispatchers are stopped.
        // Create the cache dispatcher and start it.
        mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
        mCacheDispatcher.start();

        // Create network dispatchers (and corresponding threads) up to the pool size.
        for (int i = 0; i &lt; mDispatchers.length; i++) {
            NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
                    mCache, mDelivery);
            mDispatchers[i] = networkDispatcher;
            networkDispatcher.start();
        }
    }

```

此处主要是mCacheDispatcher和NetworkDispatcher，他们都是继承的Thread类。mCacheDispatcher是缓存调度的线程，NetworkDispatcher是网络调度的线程。他们的具体实现等会再说，我们先看RequestQueue.start（）中的逻辑。

首先，我们可以看到新建了一个mCacheDispatcher对象，并且调用了start（）方法。 然后根据mDispatchers的长度新建了几个NetworkDispatcher对象并且开启。具体的mDispatchers的长度，我们可以在源码中找到：

```
 public RequestQueue(Cache cache, Network network, int threadPoolSize,
            ResponseDelivery delivery) {
        mCache = cache;
        mNetwork = network;
        mDispatchers = new NetworkDispatcher[threadPoolSize];
        mDelivery = delivery;
    }

```

```
 public RequestQueue(Cache cache, Network network) {
        this(cache, network, DEFAULT_NETWORK_THREAD_POOL_SIZE);
    }

```

```
    /** Number of network request dispatcher threads to start. */
    private static final int DEFAULT_NETWORK_THREAD_POOL_SIZE = 4;

```

可以看到在默认情况下，NetworkDispatcher也就是网络调度线程会开启4个，当然我们也可以自定义开启的个数。

>  
 所以默认是开启5个线程，即5个线程并发，1个缓存调度线程，4个网络调度线程。 


然后我们就看一下NetworkDispatcher和CacheDispatcher分别干了什么吧。 #网络调度线程：NetworkDispatcher

```
@Override
    public void run() {
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        while (true) {
            long startTimeMs = SystemClock.elapsedRealtime();
            Request&lt;?&gt; request;
            try {
                // Take a request from the queue.
                request = mQueue.take();
            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }

            try {
                request.addMarker("network-queue-take");

                // If the request was cancelled already, do not perform the
                // network request.
                if (request.isCanceled()) {
                    request.finish("network-discard-cancelled");
                    continue;
                }

                addTrafficStatsTag(request);

                // Perform the network request.
                NetworkResponse networkResponse = mNetwork.performRequest(request);
                request.addMarker("network-http-complete");

                // If the server returned 304 AND we delivered a response already,
                // we're done -- don't deliver a second identical response.
                if (networkResponse.notModified &amp;&amp; request.hasHadResponseDelivered()) {
                    request.finish("not-modified");
                    continue;
                }

                // Parse the response here on the worker thread.
                Response&lt;?&gt; response = request.parseNetworkResponse(networkResponse);
                request.addMarker("network-parse-complete");

                // Write to cache if applicable.
                // TODO: Only update cache metadata instead of entire record for 304s.
                if (request.shouldCache() &amp;&amp; response.cacheEntry != null) {
                    mCache.put(request.getCacheKey(), response.cacheEntry);
                    request.addMarker("network-cache-written");
                }

                // Post the response back.
                request.markDelivered();
                mDelivery.postResponse(request, response);
            } catch (VolleyError volleyError) {
                volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
                parseAndDeliverNetworkError(request, volleyError);
            } catch (Exception e) {
                VolleyLog.e(e, "Unhandled exception %s", e.toString());
                VolleyError volleyError = new VolleyError(e);
                volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
                mDelivery.postError(request, volleyError);
            }
        }
    }

```

主要逻辑： 1、让线程优先级低于一般的，可以减少对用户界面的影响。 2、启动后会不断从网络请求队列中取请求处理，队列为空则等待 3、通过performRequest()，取到请求后会执行,并返回执行结果，这里的请求就是我们之前定义的StringRequest，JsonRequest等等。 4、通过parseNetworkResponse(),会解析返回的请求结果，并且返回解析后的结果。 5、解析结束后，查看结果是否需要缓存并且是否已经缓存，如果需要缓存，并且没有缓存的，那么就缓存。 6、如果是针对数据量大，并且访问不频繁甚至一次的请求，我们可以设置不需要缓存，在这里就不会缓存。 7、最终把结果传递给ResponseDelivery去执行后续处理。

#缓存调度线程：CacheDispatcher

```
 @Override
    public void run() {
        if (DEBUG) VolleyLog.v("start new dispatcher");
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

        // Make a blocking call to initialize the cache.
        mCache.initialize();

        while (true) {
            try {
                // Get a request from the cache triage queue, blocking until
                // at least one is available.
                final Request&lt;?&gt; request = mCacheQueue.take();
                request.addMarker("cache-queue-take");

                // If the request has been canceled, don't bother dispatching it.
                if (request.isCanceled()) {
                    request.finish("cache-discard-canceled");
                    continue;
                }

                // Attempt to retrieve this item from cache.
                Cache.Entry entry = mCache.get(request.getCacheKey());
                if (entry == null) {
                    request.addMarker("cache-miss");
                    // Cache miss; send off to the network dispatcher.
                    mNetworkQueue.put(request);
                    continue;
                }

                // If it is completely expired, just send it to the network.
                if (entry.isExpired()) {
                    request.addMarker("cache-hit-expired");
                    request.setCacheEntry(entry);
                    mNetworkQueue.put(request);
                    continue;
                }

                // We have a cache hit; parse its data for delivery back to the request.
                request.addMarker("cache-hit");
                Response&lt;?&gt; response = request.parseNetworkResponse(
                        new NetworkResponse(entry.data, entry.responseHeaders));
                request.addMarker("cache-hit-parsed");

                if (!entry.refreshNeeded()) {
                    // Completely unexpired cache hit. Just deliver the response.
                    mDelivery.postResponse(request, response);
                } else {
                    // Soft-expired cache hit. We can deliver the cached response,
                    // but we need to also send the request to the network for
                    // refreshing.
                    request.addMarker("cache-hit-refresh-needed");
                    request.setCacheEntry(entry);

                    // Mark the response as intermediate.
                    response.intermediate = true;

                    // Post the intermediate response back to the user and have
                    // the delivery then forward the request along to the network.
                    mDelivery.postResponse(request, response, new Runnable() {
                        @Override
                        public void run() {
                            try {
                                mNetworkQueue.put(request);
                            } catch (InterruptedException e) {
                                // Not much we can do about this.
                            }
                        }
                    });
                }

            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }
        }
    }

```

主要逻辑： 1、不断从缓存请求队列中取请求处理，队列为空则等待 3、当结果未缓存过、缓存失效或缓存需要刷新的情况下，就将该请求放入网络请求队列，在NetworkDispatcher中进行调度处理。 2、如果获取到了缓存，缓存请求处理结果传递给ResponseDelivery去执行后续处理。 #缓存数据结构 在上面两个调度线程中，我们可以看到缓存对象是mCache，一层一层向上找，我们最终可以在newRequestQueue看到这行代码：

```
RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);

```

DiskBasedCache就是Volley中该缓存的数据结构:

```
public class DiskBasedCache implements Cache {

    /** Map of the Key, CacheHeader pairs */
    private final Map&lt;String, CacheHeader&gt; mEntries =
            new LinkedHashMap&lt;String, CacheHeader&gt;(16, .75f, true);

```

仅仅看前几行，我们就可以知道它是通过LinkedHashMap这个双向链表进行缓存的。初始长度为16，负载因子为0.75。

#在主线程响应：ResponseDelivery NetworkDispatcher和CacheDispatcher最终都将解析后的数据交给了mDelivery处理，那么这是什么呢？最终我们可以发现是在RequestQueue这个方法中定义的，如下：

```
 public RequestQueue(Cache cache, Network network, int threadPoolSize) {
        this(cache, network, threadPoolSize,
                new ExecutorDelivery(new Handler(Looper.getMainLooper())));
    }
    public RequestQueue(Cache cache, Network network) {
        this(cache, network, DEFAULT_NETWORK_THREAD_POOL_SIZE);
    }

```

我们调用的时候使用的是后一个方法，但是会跳转到第一个，于是我们知道mDelivery其实就是一个ExecutorDelivery对象，于是我们来看下它的源码：

```
public class ExecutorDelivery implements ResponseDelivery {
    /** Used for posting responses, typically to the main thread. */
    private final Executor mResponsePoster;

    /**
     * Creates a new response delivery interface.
     * @param handler {@link Handler} to post responses on
     */
    public ExecutorDelivery(final Handler handler) {
        // Make an Executor that just wraps the handler.
        mResponsePoster = new Executor() {
            @Override
            public void execute(Runnable command) {
                handler.post(command);
            }
        };
    }

    /**
     * Creates a new response delivery interface, mockable version
     * for testing.
     * @param executor For running delivery tasks
     */
    public ExecutorDelivery(Executor executor) {
        mResponsePoster = executor;
    }

    @Override
    public void postResponse(Request&lt;?&gt; request, Response&lt;?&gt; response) {
        postResponse(request, response, null);
    }

    @Override
    public void postResponse(Request&lt;?&gt; request, Response&lt;?&gt; response, Runnable runnable) {
        request.markDelivered();
        request.addMarker("post-response");
        mResponsePoster.execute(new ResponseDeliveryRunnable(request, response, runnable));
    }

    @Override
    public void postError(Request&lt;?&gt; request, VolleyError error) {
        request.addMarker("post-error");
        Response&lt;?&gt; response = Response.error(error);
        mResponsePoster.execute(new ResponseDeliveryRunnable(request, response, null));
    }

    /**
     * A Runnable used for delivering network responses to a listener on the
     * main thread.
     */
    @SuppressWarnings("rawtypes")
    private class ResponseDeliveryRunnable implements Runnable {
        private final Request mRequest;
        private final Response mResponse;
        private final Runnable mRunnable;

        public ResponseDeliveryRunnable(Request request, Response response, Runnable runnable) {
            mRequest = request;
            mResponse = response;
            mRunnable = runnable;
        }

        @SuppressWarnings("unchecked")
        @Override
        public void run() {
            // If this request has canceled, finish it and don't deliver.
            if (mRequest.isCanceled()) {
                mRequest.finish("canceled-at-delivery");
                return;
            }

            // Deliver a normal response or error, depending.
            if (mResponse.isSuccess()) {
                mRequest.deliverResponse(mResponse.result);
            } else {
                mRequest.deliverError(mResponse.error);
            }

            // If this is an intermediate response, add a marker, otherwise we're done
            // and the request can be finished.
            if (mResponse.intermediate) {
                mRequest.addMarker("intermediate-response");
            } else {
                mRequest.finish("done");
            }

            // If we have been provided a post-delivery runnable, run it.
            if (mRunnable != null) {
                mRunnable.run();
            }
       }
    }
}

```

源码虽多，但是功能几句话就可以描述： 它会在 Handler 对应线程中传输缓存调度线程或者网络调度线程中产生的请求结果或请求错误，会在请求成功的情况下调用 Request.deliverResponse(…) 函数，失败时调用 Request.deliverError(…) 函数。

当我们去查看deliverResponse和deliverError的时候，可以发现他们都是Request中的方法，然后我们可以把JsonRequest当做例子看一下源码：

```
public abstract class JsonRequest&lt;T&gt; extends Request&lt;T&gt; {
    /** Default charset for JSON request. */
    protected static final String PROTOCOL_CHARSET = "utf-8";

    /** Content type for request. */
    private static final String PROTOCOL_CONTENT_TYPE =
        String.format("application/json; charset=%s", PROTOCOL_CHARSET);

    private final Listener&lt;T&gt; mListener;
    private final String mRequestBody;

    /**
     * Deprecated constructor for a JsonRequest which defaults to GET unless {@link #getPostBody()}
     * or {@link #getPostParams()} is overridden (which defaults to POST).
     *
     * @deprecated Use {@link #JsonRequest(int, String, String, Listener, ErrorListener)}.
     */
    public JsonRequest(String url, String requestBody, Listener&lt;T&gt; listener,
            ErrorListener errorListener) {
        this(Method.DEPRECATED_GET_OR_POST, url, requestBody, listener, errorListener);
    }

    public JsonRequest(int method, String url, String requestBody, Listener&lt;T&gt; listener,
            ErrorListener errorListener) {
        super(method, url, errorListener);
        mListener = listener;
        mRequestBody = requestBody;
    }

    @Override
    protected void deliverResponse(T response) {
        mListener.onResponse(response);
    }

    @Override
    abstract protected Response&lt;T&gt; parseNetworkResponse(NetworkResponse response);

    /**
     * @deprecated Use {@link #getBodyContentType()}.
     */
    @Override
    public String getPostBodyContentType() {
        return getBodyContentType();
    }

    /**
     * @deprecated Use {@link #getBody()}.
     */
    @Override
    public byte[] getPostBody() {
        return getBody();
    }

    @Override
    public String getBodyContentType() {
        return PROTOCOL_CONTENT_TYPE;
    }

    @Override
    public byte[] getBody() {
        try {
            return mRequestBody == null ? null : mRequestBody.getBytes(PROTOCOL_CHARSET);
        } catch (UnsupportedEncodingException uee) {
            VolleyLog.wtf("Unsupported Encoding while trying to get the bytes of %s using %s",
                    mRequestBody, PROTOCOL_CHARSET);
            return null;
        }
    }
}

```

源码虽多，重点就一个，如下：

```

    public JsonRequest(int method, String url, String requestBody, Listener&lt;T&gt; listener,
            ErrorListener errorListener) {
        super(method, url, errorListener);
        mListener = listener;
        mRequestBody = requestBody;
    }

    @Override
    protected void deliverResponse(T response) {
        mListener.onResponse(response);
    }

```

我们在ExecutorDelivery中，在请求成功时调用的deliverResponse（）方法，其实就是我们开始定义Request（如JsonRequest、StringRequest等）中所定义的Listener.onResponse（）方法。

如此其实Volley源码的解析就基本结束了。

#总结：

最终我们再根据Volley请求流程图概括一下： <img src="https://img-blog.csdn.net/20170428084245381?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述">

##流程： 1、调用RequestQueue的add()方法来添加一条请求。 2、请求会先被加入到缓存队列当中，缓存调度线程会从中获取请求。如果获取到了，就会解析并且返回主线程响应。如果获取不到，就会将这条请求放入网络请求队列。 3、网络调度线程会从网络请求队列获取到请求然后处理，发送HTTP请求，解析响应结果，写入缓存，返回主线程响应。

##要点： 1、缓存调度线程只有一个，不可自定义. 2、网络调度线程默认有4个，可以自定义个数。 3、缓存调度线程和网络调度线程都是使用while（true）不断从队列中读取请求，但是不是死循环，当队列中没有请求的时候，会等待。 4、Volley是默认所有请求都会缓存的，但是针对数据量大，并且访问不频繁甚至一次的请求，我们可以设置不需要缓存。 5、缓存使用的数据结构是双向链表LinkedHashmap。