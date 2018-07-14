## Android Question List

> 基础

### 1.Activity生命周期

* 正常情况下的生命周期
* 复用Activity情况下的生命周期
* Activity异常销毁时的生命周期

* onNewIntent，在Activity启动模式为SingleTop/SingleTask/SingleInstance时，复用Activity时，会调用onNewIntent方法

### 2.Activity启动模式（LaunchMode）

### 3.Handler 、MessageQueue 、Looper三者的关系和原理；

### AsyncTask 、HanlderThread 、IntentService 的原理和使用场景；

### IPC通信机制

1. Android进程间通信（IPC）的实现方式：Bundle，文件共享，AIDL，Messenger，ContentProvider，Socket，BroadCastReceiver
2. Android进程间通信（IPC）的机制：Binder Parcelable（此处不在叙述Serializable）Parcelable接口在完成数据的序列化过程之后，通过Binder进行传输。实质上Parcelable包含序列化和反序列化

- 那么问题还是来了，为啥需要多进程？如何实现多进程？
  - 为啥要多进程？
    1. Android对单个应用所使用的最大内存做了限制。
    2. 在不同的应用之间共享数 据。
- 如何实现多进程？Manifest清单文件中声明process属性
- WebView开发的时候为啥一般会指定独立进程？ 
  - WebView导致的OOM问题
  - Android版本不同，采用了不同的内核，兼容性Crash
  - WebView代码质量，WebView和Native版本不一致，导致Crash
- 最近两年的开发没有使用到AIDL和Binder，知识点记住，面试要能答上来，具体细节可以在使用的时候查询。  

### 

> View

### Android View体系

1. View的工作原理

   1. onDraw主体绘制

2. 事件分发机制

   1. 关于事件分发机制，可以从两个角度讲解，其一是事件分发的原理和流程

   其实质上，是从Activity向下逐层递归调用dispatchTouchEvent()方法，dispatchTouchEvent()本身是一个调度方法，其内部调用onInterceptTouchEvent，判断当前View是否要拦截该事件，如果不拦截，接着调用onTouchEvent，将事件向下分发，直到有子View返回true，消费该事件。

   其中，onTouchEvent中，如果子View在该事件的action.DOWN事件返回了false，那么本次触摸以后的事件子View都不会收到

   1. 其二是处理事件冲突的具体思路

   具体思路有三个：

   1. 实现自定义触摸反馈，直接在onTouchEvent中处理事件就OK
   2. 父View想要事件，直接在onInterceptTouchEvent中返回true进行拦截。此时，父View对给子View分发一个action.CANCEL的事件，通知子View恢复状态。
   3. 子View不想要父View拦截，调用requestDisallowInterceptTouchEvent()方法，请求父View不要拦截，该方法只对本地事件有效。

   

3. 动画（主要是属性动画）这块的内容从hencoder网站及笔记复习

4. 自定义View/ViewGroup实现

   1. 自定义View已经基本掌握（精通），自定义ViewGroup的关键在于精通onMeasure中关于测量的方法和MeasureSpec的三种不同的模式，onLayout只是简单地把子View布置到该放置的地方，因此onMeasure是自定义ViewGroup的精髓所在。
   2. 要精准地理解View绘制的流程/方法/概念。

### RecyclerView 和 ListView 的相同和不同点，在 item 回收上有什么不同；

View 事件分发机制；

### 3.Window View Activity三者的关系，追问Acitivty如何和Window关联

Android Framework层有没有了解过，说说 Window 窗口添加的过程；

### 6.自定义View的具体过程

### 7.onMeasure的具体过程，先measure子view还是自己

### 8.onDraw的具体过程，先draw子view还是自己



> 热点技术

### .热修复

### 10.模块化

### 11.组件化的原理，还有一些组件化平时使用的问题；

消息推送有没有做过，推送到达率的问题

> 架构

说说 apk 打包流程以及多渠道打包；

### MVC MVP MVVM的理解

### 应用程序崩溃统计以及数据分析





## 开源框架

### 1. OKHttp 责任链模式
### 2. Glide 图片加载的三级缓存机制 以及 Bitmap 优化
### 3. RxJava flatMap和Map的作用

### 4. Retrofit

### 5.什么是依赖注入？能说几个依赖注入的库么？你使用过哪些？



