## Other Question List（Network|Structure）

### 1.Gradle 的 api 和 implementation 有什么区别

区别就是是否将依赖暴露出去。api 会暴露，implementation 不会。

使用 implementation 时，依赖库变动的话只会影响、重新编译到当前库，不会影响到其他，因此编译速度回提高。

api 依赖的库变动的话，会重新编译所有依赖它的项目，相对编译时间会变长。

### 2.HTTP 和 HTTPS的区别 以及 HTTPS协议传输数据的具体流程

[HTTPS的原理及连接流程](HTTPS的原理及连接流程.md)

### 3.HTTP 2.0 

1. HTTP/2采用二进制格式而非文本格式
2. HTTP/2是完全多路复用的，而非有序并阻塞的——只需一个连接即可实现并行，HTTP2.0使用多路复用技术(Multiplexing),多路复用允许同时通过单一的 HTTP/2 连接发起多重的请求-响应消息。
3. 使用报头压缩，HTTP/2降低了开销
4. HTTP/2让服务器可以将响应主动“推送”到客户端缓存中

### 4.网络通信相关问题，OSI七层模型，TCP/UDP的区别
