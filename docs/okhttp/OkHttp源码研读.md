# OkHttp请求网络的流程分析

在日常的开发中，OkHttp+Retrofit已经成为Android开发首选的网络请求框架。在面试中，也是必问的问题之一。今天我们就来对OkHttp的源码做一下解读。

在第一篇文章中，我们先顺蔓摸瓜，跟着方法调用的链路，分析一下okHttp网络请求的流程。后续的文章，再分别对OkHttp的核心类，OkHttpClient/Dispatcher/Interceptor等作进一步解读。

首先上图，看OkHttp的设计类图

![OkHttp设计类图](../img/okhttp_struc.png)



然后是OkHttp的请求流程图，先了解一下

![OkHttp 请求流程](../img/okhttp_full_process.png)

## OkHttpClient

先看一下我们日常异步请求的demo代码

```
val client = OkHttpClient.Builder()
          .addNetworkInterceptor(HttpLoggingInterceptor()
          .setLevel(HttpLoggingInterceptor.Level.BODY))
          .build()
val request = Request.Builder()
                .url(url)
                .build()
        client.newCall(request)
        .enqueue(object : Callback {
            override fun onFailure(call: Call?, e: IOException?) {
                error { e?.message }
            }
            override fun onResponse(call: Call?, response: Response?) {
                val result = response?.body()?.string()
                info { result }
            }
        })
```

首先我们使用OkHttpClient.Builder()初始化了一个OkHttpClient的实例，然后使用Request.Builder()构造一个Request实例，然后调用client.newCall(request) .enqueue(callback:Callback?)发出请求。下面我们顺着这个思路，一步一步去拆解。

client.newCall(request)，实际上是使用静态方法创建了一个RealCall

```
 /**
   * Prepares the {@code request} to be executed at some point in the future.
   */
  @Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
  }
```

再看enqueue()方法

```
@Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
    //当前Call实例是否已经被执行过，如果已经执行过，抛出异常
    //因此，OkHttp的每个Call，只能被执行一次
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    //捕获堆栈信息
    captureCallStackTrace();
    //Invoke listener
    eventListener.callStart(this);
    //调用Diapatcher的enqueue方法，注意这里，构造了一个AsyncCall实例
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
```

然后看Dispatcher的enqueue方法

```
  private int maxRequests = 64;//最大并发请求数量
  private int maxRequestsPerHost = 5;//单个Host的最大并发请求数量
  
  /** Ready async calls in the order they'll be run. */
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  /** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  synchronized void enqueue(AsyncCall call) {
  //判断一下，如果当前总的请求数不超过最大并发数/单个Host的最大并发请求数量
  //runningCallsForHost(call)遍历了队列中的所有AsyncCall，找过和当前host相同的AsyncCall的数量
      if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) <    maxRequestsPerHost) {
      //加入队列
      runningAsyncCalls.add(call);
      //线程池执行异步请求
      executorService().execute(call);
    } else {
    //加入等待队列，那么问题来了，readyAsyncCalls里面的请求是如何被执行的呢？
    //先把问题抛出来，后面整个流程看完，会水到渠成地get这个问题的答案
      readyAsyncCalls.add(call);
    }
  }
```

Ok,到这一步已经很明朗了，真正的网络请求逻辑应该在AsyncCall的run方法里面，跑不了了。

打开AsyncCall发现，AsyncCall实现了NamedRunnable接口

```
final class AsyncCall extends NamedRunnable
```

```
public abstract class NamedRunnable implements Runnable {
  protected final String name;

  public NamedRunnable(String format, Object... args) {
    this.name = Util.format(format, args);
  }

  @Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
}
```

再看NamedRunnable，真是名副其实，就是给当前的线程setName，把具体执行逻辑扔到了execute()方法中了，好吧，我们接着回到AsyncCall

```
//构造方法中，super调用父类，设定当前线程的名字
//从Dispatcher的源码中可以看到，ThreadFactory在初始化线程的时候，name都是"OkHttp Dispatcher"
//在这里，会根据请求地址修改线程名字，方便调试
    AsyncCall(Callback responseCallback) {
      super("OkHttp %s", redactedUrl());
      this.responseCallback = responseCallback;
    }
    
    //大招来了
    //一层一层地拨开云雾，终于找到你了
    //这里关注两个方法getResponseWithInterceptorChain()和 client.dispatcher().finished(this)
    @Override protected void execute() {
      boolean signalledCallback = false;
      try {
      //初看源码的时候，以为这里只是拦截请求，并作一些处理，苦于找不到请求的入口，而放弃了。
      //这次终于被我抓住了。
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
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
      //上文中我们抛出过一个问题，readyAsyncCalls队列中的AsyncCall是怎么被执行的
      //答案就在这里
        client.dispatcher().finished(this);
      }
    }
  }

```
这里我们先看一下client.dispatcher().finished(this)的实现
```
 /** Used by {@code AsyncCall#run} to signal completion. */
  void finished(AsyncCall call) {
    finished(runningAsyncCalls, call, true);
  }
  
  private <T> void finished(Deque<T> calls, T call, boolean promoteCalls) {
    int runningCallsCount;
    Runnable idleCallback;
    synchronized (this) {
    //移除当前的call
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      //这里不再展开，只描述一下promoteCalls()的作用
      //迭代器迭代readyAsyncCalls，拿出next的AsyncCall，加入runningAsyncCalls，然后线程池去执行
      if (promoteCalls) promoteCalls();
      runningCallsCount = runningCallsCount();
      idleCallback = this.idleCallback;
    }

    if (runningCallsCount == 0 && idleCallback != null) {
      idleCallback.run();
    }
  }
```

因为finish()是在finally代码块中执行的，也就是说，无论当前的AsyncCall请求成功/失败，都会被移除出队列，然后下一个等待任务进入队列，被执行。

接着回到主线

```
Response response = getResponseWithInterceptorChain();
```

这段代码可谓是精髓所在了，方法实现

```
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    //client.interceptors()里面是我们在构造OkHttpClient.Builder()对象时，使用addInterceptor添加的拦截器
    interceptors.addAll(client.interceptors());
    //负责失败重试以及重定向
    interceptors.add(retryAndFollowUpInterceptor);
    //负责对用户请求作封装，转换成可以发送到服务器的请求
    //以及对服务器的响应作封装，转换成用户友好的响应
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    //负责缓存逻辑处理
    interceptors.add(new CacheInterceptor(client.internalCache()));
    //负责和服务器建立连接
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
    //client.interceptors()里面是我们在构造OkHttpClient.Builder()对象时，使用addNetworkInterceptor添加的拦截器
      interceptors.addAll(client.networkInterceptors());
    }
    //负责向服务器发送请求
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
  }
```

[责任链模式](https://zh.wikipedia.org/wiki/%E8%B4%A3%E4%BB%BB%E9%93%BE%E6%A8%A1%E5%BC%8F)在这个 `Interceptor` 链条中得到了很好的实践（感谢 Stay 一语道破，自愧弗如）。

> 它包含了一些命令对象和一系列的处理对象，每一个处理对象决定它能处理哪些命令对象，它也知道如何将它不能处理的命令对象传递给该链中的下一个处理对象。该模式还描述了往该处理链的末尾添加新的处理对象的方法。

接着看RealInterceptorChain.proceed()

```
 public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      RealConnection connection) throws IOException {
    if (index >= interceptors.size()) throw new AssertionError();

    calls++;

    // If we already have a stream, confirm that the incoming request will use it.
    if (this.httpCodec != null && !this.connection.supportsUrl(request.url())) {
      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
          + " must retain the same host and port");
    }

    // If we already have a stream, confirm that this is the only call to chain.proceed().
    if (this.httpCodec != null && calls > 1) {
      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
          + " must call proceed() exactly once");
    }

    // Call the next interceptor in the chain.
    //这里构造出下一个Interceptor
    RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
        connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
        writeTimeout);
    Interceptor interceptor = interceptors.get(index);
    //调用自己的Interceptor方法
    Response response = interceptor.intercept(next);

    // Confirm that the next interceptor made its required call to chain.proceed().
    if (httpCodec != null && index + 1 < interceptors.size() && next.calls != 1) {
      throw new IllegalStateException("network interceptor " + interceptor
          + " must call proceed() exactly once");
    }

    // Confirm that the intercepted response isn't null.
    if (response == null) {
      throw new NullPointerException("interceptor " + interceptor + " returned null");
    }

    if (response.body() == null) {
      throw new IllegalStateException(
          "interceptor " + interceptor + " returned a response with no body");
    }

    return response;
  }
```

这个方法时责任链模式的核心，结合上面的代码，Interceptor的调用顺序如下：

![Interceptor责任链调用顺序](../img/interceptor_chain.png)

CallServerInterceptor位于链路的末端，是具体向服务器发送请求并接受响应的拦截器。

OK，到此为止，我们把OkHttp异步网络请求的流程梳理清楚了。同步请求，相比异步请求，更为简单

```
@Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    try {
    //加入runningSyncCalls队列
      client.dispatcher().executed(this);
      //同异步请求
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } catch (IOException e) {
      eventListener.callFailed(this, e);
      throw e;
    } finally {
      client.dispatcher().finished(this);
    }
  }
  
   /** Used by {@code Call#execute} to signal it is in-flight. */
  synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }
```