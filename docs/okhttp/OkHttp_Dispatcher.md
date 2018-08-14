## Dispatcher源码解读

```
/**
 * Policy on when async requests are executed.
 * 执行异步请求的策略
 * 每个Dispatcher内部有一个ExecutorService线程池
 * <p>Each dispatcher uses an {@link ExecutorService} to run calls internally. If you supply your
 * own executor, it should be able to run {@linkplain #getMaxRequests the configured maximum} number
 * of calls concurrently.
 */
```

上面我们已经看过enqueue方法的实现，再看一下

```
synchronized void enqueue(AsyncCall call) {
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);
      executorService().execute(call);
    } else {
      readyAsyncCalls.add(call);
    }
  }
```

接着跟踪executorService()方法

```
public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }
```

返回一个线程池，该线程池没有核心线程，最大线程数Integer.MAX_VALUE，线程空闲时间超过60s就会被回收，new SynchronousQueue<Runnable>()用来维护线程队列，看完之后发现，这不就是一个CachedThreadPool么？只不过ThreadFractory中声明线程名称为"OkHttp Dispatcher"。