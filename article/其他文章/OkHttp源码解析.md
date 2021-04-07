#OkHttp源码解析
# 同步请求用例

```
//创建对象
OkHttpClient client = new OkHttpClient();
//创建请求
Request request = new Request.Builder()
		.url("http://blog.csdn.net/double2hao")
        .build();
//网络获取
Response response = client.newCall(request).execute();

```

# OkHttpClient 构造

```
public OkHttpClient() {
  this(new Builder());
}

```

```
public Builder() {
  dispatcher = new Dispatcher();
  protocols = DEFAULT_PROTOCOLS;
  connectionSpecs = DEFAULT_CONNECTION_SPECS;
  proxySelector = ProxySelector.getDefault();
  cookieJar = CookieJar.NO_COOKIES;
  socketFactory = SocketFactory.getDefault();
  hostnameVerifier = OkHostnameVerifier.INSTANCE;
  certificatePinner = CertificatePinner.DEFAULT;
  proxyAuthenticator = Authenticator.NONE;
  authenticator = Authenticator.NONE;
  connectionPool = new ConnectionPool();
  dns = Dns.SYSTEM;
  followSslRedirects = true;
  followRedirects = true;
  retryOnConnectionFailure = true;
  connectTimeout = 10_000;
  readTimeout = 10_000;
  writeTimeout = 10_000;
}

```

可以看到，这里逻辑很简单，使用建造者模式，让OkHttpClient 中的参数都使用默认的配置。

# 网络获取步骤

在创建OkHttpClient 对象和Request对象之后，调用OkHttpClient .newCall(方法)，新建一个RealCall，然，并且调用它的execute()方法，我们可以看下源码。

```
  @Override public Call newCall(Request request) {
    return new RealCall(this, request, false /* for web socket */);
  }

```

```
  RealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    final EventListener.Factory eventListenerFactory = client.eventListenerFactory();

    this.client = client;
    this.originalRequest = originalRequest;
    this.forWebSocket = forWebSocket;
    this.retryAndFollowUpInterceptor = new RetryAndFollowUpInterceptor(client, forWebSocket);

    // TODO(jwilson): this is unsafe publication and not threadsafe.
    this.eventListener = eventListenerFactory.create(this);
  }

```

RealCall的构造中也仅仅是为内部参数设置了默认的配置。 于是关键定然就是在RealCall.execute()中了。

```
    @Override protected void execute() {
      boolean signalledCallback = false;
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }

```

此处的逻辑其实也是比较简单，就是尝试通过getResponseWithInterceptorChain()方法获取到response。我们看一下这个方法的源码：

```
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List&lt;Interceptor&gt; interceptors = new ArrayList&lt;&gt;();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }

```

这里在interceptors 中加入了一个个Interceptor，最终将Interceptors和Request传入了RealInterceptorChain执行。

```
  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      RealConnection connection) throws IOException {
    if (index &gt;= interceptors.size()) throw new AssertionError();

    calls++;

    // If we already have a stream, confirm that the incoming request will use it.
    if (this.httpCodec != null &amp;&amp; !this.connection.supportsUrl(request.url())) {
      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
          + " must retain the same host and port");
    }

    // If we already have a stream, confirm that this is the only call to chain.proceed().
    if (this.httpCodec != null &amp;&amp; calls &gt; 1) {
      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
          + " must call proceed() exactly once");
    }

    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(
        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);

    // Confirm that the next interceptor made its required call to chain.proceed().
    if (httpCodec != null &amp;&amp; index + 1 &lt; interceptors.size() &amp;&amp; next.calls != 1) {
      throw new IllegalStateException("network interceptor " + interceptor
          + " must call proceed() exactly once");
    }

    // Confirm that the intercepted response isn't null.
    if (response == null) {
      throw new NullPointerException("interceptor " + interceptor + " returned null");
    }

    return response;
  }

```

RealInterceptorChain方法中忽略了各种异常情况后，可以发现它其实就是一个递归。它会根据存入顺序将interceptors中的interceptor的intercept()的方法全部执行一遍。 但是如果遍历途中已经获得了response，那么就会返回。

# Interceptor 的责任链模式

关于此处Interceptor的逻辑，在下图中可以很好的体现。 （图片转自：https://blog.piasy.com/2016/07/11/Understand-OkHttp/） <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/1860.png" alt="这里写图片描述">

**责任链模式概念** 多个对象都有机会处理请求，从而避免了请求的发送者和接收者之间的耦合关系。将这些对象连城一条链，并沿着这条链传递该请求，直到把它处理完为止。

知道了责任链模式的概念，这里就很好理解了，直接讲一下每一个Interceptor的大概作用。
- **RetryAndFollowUpInterceptor**：负责失败重试以及重定向。- **BridgeInterceptor**：负责把用户构造的请求转换为发送到服务器的请求（主要会在请求中加入一些header）、把服务器返回的响应转换为用户友好的响应。- **CacheInterceptor**：负责读取缓存直接返回、更新缓存。- **ConnectInterceptor**：负责和服务器建立连接。- **CallServerInterceptor**：负责向服务器发送请求数据、从服务器读取响应数据。
接下来从源码角度分别看一下各个Interceptor具体做的事情。

# Interceptor源码

## RetryAndFollowUpInterceptor

```
  @Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();

    streamAllocation = new StreamAllocation(
        client.connectionPool(), createAddress(request.url()), callStackTrace);

    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      if (canceled) {
        streamAllocation.release();
        throw new IOException("Canceled");
      }

      Response response = null;
      boolean releaseConnection = true;
      try {
        response = ((RealInterceptorChain) chain).proceed(request, streamAllocation, null, null);
        releaseConnection = false;
      } catch (RouteException e) {
        // The attempt to connect via a route failed. The request will not have been sent.
        if (!recover(e.getLastConnectException(), false, request)) {
          throw e.getLastConnectException();
        }
        releaseConnection = false;
        continue;
      } catch (IOException e) {
        // An attempt to communicate with a server failed. The request may have been sent.
        boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
        if (!recover(e, requestSendStarted, request)) throw e;
        releaseConnection = false;
        continue;
      } finally {
        // We're throwing an unchecked exception. Release any resources.
        if (releaseConnection) {
          streamAllocation.streamFailed(null);
          streamAllocation.release();
        }
      }

      // Attach the prior response if it exists. Such responses never have a body.
      if (priorResponse != null) {
        response = response.newBuilder()
            .priorResponse(priorResponse.newBuilder()
                    .body(null)
                    .build())
            .build();
      }

      Request followUp = followUpRequest(response);

      if (followUp == null) {
        if (!forWebSocket) {
          streamAllocation.release();
        }
        return response;
      }

      closeQuietly(response.body());

      if (++followUpCount &gt; MAX_FOLLOW_UPS) {
        streamAllocation.release();
        throw new ProtocolException("Too many follow-up requests: " + followUpCount);
      }

      if (followUp.body() instanceof UnrepeatableRequestBody) {
        streamAllocation.release();
        throw new HttpRetryException("Cannot retry streamed HTTP body", response.code());
      }

      if (!sameConnection(response, followUp.url())) {
        streamAllocation.release();
        streamAllocation = new StreamAllocation(
            client.connectionPool(), createAddress(followUp.url()), callStackTrace);
      } else if (streamAllocation.codec() != null) {
        throw new IllegalStateException("Closing the body of " + response
            + " didn't close its backing stream. Bad interceptor?");
      }

      request = followUp;
      priorResponse = response;
    }
  }

```
1. 创建了streamAllocation对象，负责管理连接、流和请求三者之间的关系。1. 调用传进来的chain参数尝试获取响应，如果获取失败了，在各个异常中都会调用recover方法尝试恢复请求。1. 如果获取response，通过response获取followUp的重定向请求，如果不存在重定向请求，则返回response。如果存在重定向请求，则让当前请求等于这个重定向请求，继续while的循环。1. 其中通过chain获取响应会进入BridgeInterceptor。
## BridgeInterceptor

```
  @Override public Response intercept(Chain chain) throws IOException {
    Request userRequest = chain.request();
    Request.Builder requestBuilder = userRequest.newBuilder();

    RequestBody body = userRequest.body();
    if (body != null) {
      MediaType contentType = body.contentType();
      if (contentType != null) {
        requestBuilder.header("Content-Type", contentType.toString());
      }

      long contentLength = body.contentLength();
      if (contentLength != -1) {
        requestBuilder.header("Content-Length", Long.toString(contentLength));
        requestBuilder.removeHeader("Transfer-Encoding");
      } else {
        requestBuilder.header("Transfer-Encoding", "chunked");
        requestBuilder.removeHeader("Content-Length");
      }
    }

    if (userRequest.header("Host") == null) {
      requestBuilder.header("Host", hostHeader(userRequest.url(), false));
    }

    if (userRequest.header("Connection") == null) {
      requestBuilder.header("Connection", "Keep-Alive");
    }

    // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
    // the transfer stream.
    boolean transparentGzip = false;
    if (userRequest.header("Accept-Encoding") == null &amp;&amp; userRequest.header("Range") == null) {
      transparentGzip = true;
      requestBuilder.header("Accept-Encoding", "gzip");
    }

    List&lt;Cookie&gt; cookies = cookieJar.loadForRequest(userRequest.url());
    if (!cookies.isEmpty()) {
      requestBuilder.header("Cookie", cookieHeader(cookies));
    }

    if (userRequest.header("User-Agent") == null) {
      requestBuilder.header("User-Agent", Version.userAgent());
    }

    Response networkResponse = chain.proceed(requestBuilder.build());

    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());

    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);

    if (transparentGzip
        &amp;&amp; "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        &amp;&amp; HttpHeaders.hasBody(networkResponse)) {
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      responseBuilder.body(new RealResponseBody(strippedHeaders, Okio.buffer(responseBody)));
    }

    return responseBuilder.build();
  }

```
1. 首先获取原请求，然后在请求中添加头，比如Host、Connection、Accept-Encoding参数等，然后根据看是否需要填充Cookie。1. 使用chain的procced方法得到响应1. 获取响应后，对响应做处理得到用户响应,最后返回。1. 使用chain的procced方法获取响应会进入CacheInterceptor。
## CacheInterceptor

```
  @Override public Response intercept(Chain chain) throws IOException {
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();

    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    if (cache != null) {
      cache.trackResponse(strategy);
    }

    if (cacheCandidate != null &amp;&amp; cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.
    if (networkRequest == null &amp;&amp; cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    Response networkResponse = null;
    try {
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null &amp;&amp; cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.
    if (cacheResponse != null) {
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache.trackConditionalCacheHit();
        cache.update(cacheResponse, response);
        return response;
      } else {
        closeQuietly(cacheResponse.body());
      }
    }

    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (cache != null) {
      if (HttpHeaders.hasBody(response) &amp;&amp; CacheStrategy.isCacheable(response, networkRequest)) {
        // Offer this request to the cache.
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }

    return response;
  }

```
1. 尝试从缓存中根据请求取出相应，然后创建CacheStrategy对象，该对象有两个字段networkRequest和cahceResponse，其中networkRequest不为null则表示需要进行网络请求，cacheResponse表示返回的或需要更新的缓存响应，为null则表示请求没有使用缓存。1. networkRequest == null &amp;&amp; cacheResponse == null，表示该请求不需要使用网络但是缓存响应不存在，则返回504错误的响应。1. networkRequest == null&amp;&amp; cacheResponse ！= null，表示该请求不允许使用网络，但是因为有缓存响应的存在，所以直接返回缓存响应 。。1. networkRequest ！= null，则使用chain.proceed(networkRequest)获取响应，用networkResponse 接收。1. cacheResponse != null&amp;&amp;networkResponse ==null，那么需要更新缓存，返回组合后的响应 。1. cacheResponse != null&amp;&amp;networkResponse ！=null，将组合后的响应直接写入缓存,并且返回。1. networkRequest ！= null，则使用chain.proceed(networkRequest)获取响应时，会进入ConnectInterceptor。
## ConnectInterceptor

```
  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    StreamAllocation streamAllocation = realChain.streamAllocation();

    // We need the network to satisfy this request. Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    HttpCodec httpCodec = streamAllocation.newStream(client, doExtensiveHealthChecks);
    RealConnection connection = streamAllocation.connection();

    return realChain.proceed(request, streamAllocation, httpCodec, connection);
  }

```

1、在RetryAndFollowUpInterceptor中，创建了StreamAllocation并将其传给了后面的拦截器链，所以这儿得到的StreamAllocation就是那时传入的。 2、然后获取HttpStream对象以及RealConnection对象。 3、把这些参数都传入realChain.proceed()，最终进入CallServerInterceptor。

## CallServerInterceptor

```
  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    HttpCodec httpCodec = realChain.httpStream();
    StreamAllocation streamAllocation = realChain.streamAllocation();
    RealConnection connection = (RealConnection) realChain.connection();
    Request request = realChain.request();

    long sentRequestMillis = System.currentTimeMillis();
    httpCodec.writeRequestHeaders(request);

    Response.Builder responseBuilder = null;
    if (HttpMethod.permitsRequestBody(request.method()) &amp;&amp; request.body() != null) {
      // If there's a "Expect: 100-continue" header on the request, wait for a "HTTP/1.1 100
      // Continue" response before transmitting the request body. If we don't get that, return what
      // we did get (such as a 4xx response) without ever transmitting the request body.
      if ("100-continue".equalsIgnoreCase(request.header("Expect"))) {
        httpCodec.flushRequest();
        responseBuilder = httpCodec.readResponseHeaders(true);
      }

      if (responseBuilder == null) {
        // Write the request body if the "Expect: 100-continue" expectation was met.
        Sink requestBodyOut = httpCodec.createRequestBody(request, request.body().contentLength());
        BufferedSink bufferedRequestBody = Okio.buffer(requestBodyOut);
        request.body().writeTo(bufferedRequestBody);
        bufferedRequestBody.close();
      } else if (!connection.isMultiplexed()) {
        // If the "Expect: 100-continue" expectation wasn't met, prevent the HTTP/1 connection from
        // being reused. Otherwise we're still obligated to transmit the request body to leave the
        // connection in a consistent state.
        streamAllocation.noNewStreams();
      }
    }

    httpCodec.finishRequest();

    if (responseBuilder == null) {
      responseBuilder = httpCodec.readResponseHeaders(false);
    }

    Response response = responseBuilder
        .request(request)
        .handshake(streamAllocation.connection().handshake())
        .sentRequestAtMillis(sentRequestMillis)
        .receivedResponseAtMillis(System.currentTimeMillis())
        .build();

    int code = response.code();
    if (forWebSocket &amp;&amp; code == 101) {
      // Connection is upgrading, but we need to ensure interceptors see a non-null response body.
      response = response.newBuilder()
          .body(Util.EMPTY_RESPONSE)
          .build();
    } else {
      response = response.newBuilder()
          .body(httpCodec.openResponseBody(response))
          .build();
    }

    if ("close".equalsIgnoreCase(response.request().header("Connection"))
        || "close".equalsIgnoreCase(response.header("Connection"))) {
      streamAllocation.noNewStreams();
    }

    if ((code == 204 || code == 205) &amp;&amp; response.body().contentLength() &gt; 0) {
      throw new ProtocolException(
          "HTTP " + code + " had non-zero Content-Length: " + response.body().contentLength());
    }

    return response;
  }

```

1、获取HttpStream对象，然后调用writeRequestHeaders方法写入请求的头部。 2、如果获取失败，就通过streamAllocation把流关闭。 3、如果获取成功就返回response。