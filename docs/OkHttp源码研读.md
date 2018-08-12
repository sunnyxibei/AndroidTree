##OkHttp源码研读

首先看OkHttpClient，该类的注释说明它是

```
Factory for {@linkplain Call calls}, which can be used to send HTTP requests and read their responses.
OkHttpClient实现了Call.Factory, WebSocket.Factory这两个接口。同时支持HTTP请求和WebSocket请求。
```

```
OkHttpClient()用于创建一个默认参数的Client
而其内部类OkHttpClient.Builder()用于创建一个自定义参数的Client
newBuilder()其实相当于一个深拷贝的概念，拷贝和当前相同配置的一个Client，用于一些特殊的请求。通过阅读源码发现，newBuilder返回的Client和原来的Client拥有相同的参数，包括ConnectPoll和
```

通过阅读源码发现，OkHttp支持 HTTP1.1/HTTP2.0/SPDY，支持TLS1.0-1.3，支持WebSocket

但是默认boolean isTLS = false;是关闭的

```
ConnectionSpec这个类提供了HTTPS连接时支持的cipherSuites
ConnectionSpec提供了三种模式：MODERN_TLS，COMPATIBLE_TLS，CLEARTEXT
```

```
所有Intercept onIntercept回调中的Chain实例，都是RealInterceptorChain的实例，是在RealInterceptorChain.proceed()方法中使用构造方法创建出来的，而且，这里的chain指向的是Interceptors中的下一个Interceptor
这里的处理十分的巧妙
```

```
Cookie类是OkHttp对Cookie的封装，提供的数据类，在使用CookieJar的时候，可以直接把自己Cookie的key-value放入Cookie实例中，然后把实例塞给CookieJar
```

