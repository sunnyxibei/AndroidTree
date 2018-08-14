## OkHttpClient源码解读

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
//但是这个实例和之前的实例，共享同一个diapatcher和connectionPool
public Builder newBuilder() {
    return new Builder(this);
  }
```
OkHttpClient 内部类Builder的参数
```
Builder(OkHttpClient okHttpClient) {
      //用于执行异步任务(Runnable)
      this.dispatcher = okHttpClient.dispatcher;
      this.proxy = okHttpClient.proxy;
  	  //网络协议，支持HTTP1.0/1.1/ HTTP2 SPDY3
      this.protocols = okHttpClient.protocols;
 	  //TLS版本和加密组件，HTTPS请求使用
      this.connectionSpecs = okHttpClient.connectionSpecs;
      this.interceptors.addAll(okHttpClient.interceptors);
      this.networkInterceptors.addAll(okHttpClient.networkInterceptors);
      this.eventListenerFactory = okHttpClient.eventListenerFactory;
      this.proxySelector = okHttpClient.proxySelector;
      //Cookie配置
      this.cookieJar = okHttpClient.cookieJar;
      //Cache配置
      this.internalCache = okHttpClient.internalCache;
      this.cache = okHttpClient.cache;
      //建立HTTP请求使用的socketFactory
      this.socketFactory = okHttpClient.socketFactory;
      //建立HTTPS请求使用的sslSocketFactory
      this.sslSocketFactory = okHttpClient.sslSocketFactory;
      //Use of the chain cleaner is necessary to omit 
      //unexpected certificates that aren't relevant to
 	  //the TLS handshake and to extract the trusted 
 	  //CA certificate for the benefit of certificate pinning.
 	  //看注释的意识，是在TLS handshake之前，用于清除不相关的证书
      this.certificateChainCleaner = okHttpClient.certificateChainCleaner;
      //HTTPS请求中校验host是否合法
      this.hostnameVerifier = okHttpClient.hostnameVerifier;
      this.certificatePinner = okHttpClient.certificatePinner;
      this.proxyAuthenticator = okHttpClient.proxyAuthenticator;
      this.authenticator = okHttpClient.authenticator;
      //请求池，请求复用的时候使用
      this.connectionPool = okHttpClient.connectionPool;
      this.dns = okHttpClient.dns;
      this.followSslRedirects = okHttpClient.followSslRedirects;
      this.followRedirects = okHttpClient.followRedirects;
      this.retryOnConnectionFailure = okHttpClient.retryOnConnectionFailure;
      //超时设置
      this.connectTimeout = okHttpClient.connectTimeout;
      this.readTimeout = okHttpClient.readTimeout;
      this.writeTimeout = okHttpClient.writeTimeout;
      this.pingInterval = okHttpClient.pingInterval;
```
