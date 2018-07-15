## Other Question List（Network|Structure）

### 1.Gradle 的 api 和 implementation 有什么区别

区别就是是否将依赖暴露出去。api 会暴露，implementation 不会。

使用 implementation 时，依赖库变动的话只会影响、重新编译到当前库，不会影响到其他，因此编译速度回提高。

api 依赖的库变动的话，会重新编译所有依赖它的项目，相对编译时间会变长。

### 2.http 和 https的区别 https协议传输数据的具体流程

### 3.HTTPS 握手的步骤和过程

### 4.HTTPS 的原理

### 5.HTTP 2.0 

