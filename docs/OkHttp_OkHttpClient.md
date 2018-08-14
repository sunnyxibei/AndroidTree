```
public class OkHttpClient implements Cloneable, Call.Factory, WebSocket.Factory 
```

OkHttpClient实现了Call.Factory和WebSocket.Factory，说明OkHttp同时支持HTTP请求和WebSocket请求。接口的实现如下。

```
 /**
   * Prepares the {@code request} to be executed at some point in the future.
   */
  @Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
  }

  /**
   * Uses {@code request} to connect a new web socket.
   */
  @Override public WebSocket newWebSocket(Request request, WebSocketListener listener) {
    RealWebSocket webSocket = new RealWebSocket(request, listener, new Random(), pingInterval);
    webSocket.connect(this);
    return webSocket;
  }
```

创建OkHttpClient实例的方法，有三种

```
//构造一个默认参数的OkHttpClient
public OkHttpClient() {
    this(new Builder());
  }

//我们上面使用的方式，使用Builder（建造者模式）创建实例，可以根据自己的需求修改指定对应的参数
OkHttpClient.Builder()

//使用当前的OkHttpClient实例，拷贝一个实例，然后可以对这个实例作一些修改
//但是这个实例和之前的实例，共享同一个Diapatcher，这个通过源码可以看出来
public Builder newBuilder() {
    return new Builder(this);
  }
```
OkHttpClient 内部类Builder的参数
```
Builder(OkHttpClient okHttpClient) {
      this.dispatcher = okHttpClient.dispatcher;
      this.proxy = okHttpClient.proxy;
      this.protocols = okHttpClient.protocols;
      this.connectionSpecs = okHttpClient.connectionSpecs;
      this.interceptors.addAll(okHttpClient.interceptors);
      this.networkInterceptors.addAll(okHttpClient.networkInterceptors);
      this.eventListenerFactory = okHttpClient.eventListenerFactory;
      this.proxySelector = okHttpClient.proxySelector;
      this.cookieJar = okHttpClient.cookieJar;
      this.internalCache = okHttpClient.internalCache;
      this.cache = okHttpClient.cache;
      this.socketFactory = okHttpClient.socketFactory;
      this.sslSocketFactory = okHttpClient.sslSocketFactory;
      this.certificateChainCleaner = okHttpClient.certificateChainCleaner;
      this.hostnameVerifier = okHttpClient.hostnameVerifier;
      this.certificatePinner = okHttpClient.certificatePinner;
      this.proxyAuthenticator = okHttpClient.proxyAuthenticator;
      this.authenticator = okHttpClient.authenticator;
      this.connectionPool = okHttpClient.connectionPool;
      this.dns = okHttpClient.dns;
      this.followSslRedirects = okHttpClient.followSslRedirects;
      this.followRedirects = okHttpClient.followRedirects;
      this.retryOnConnectionFailure = okHttpClient.retryOnConnectionFailure;
      this.connectTimeout = okHttpClient.connectTimeout;
      this.readTimeout = okHttpClient.readTimeout;
      this.writeTimeout = okHttpClient.writeTimeout;
      this.pingInterval = okHttpClient.pingInterval;
    }
```

