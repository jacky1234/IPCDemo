## Android 进程间通信
参考自 《Android开发艺术探索》，eclipse下突然不能执行源代码`refusing to generate code from aidl file defining parcelable	Book.aidl`，苦于国内对VPN打压，暂时不能访问Google，所以决定将项目移至AS中。

笔记参考可以参照：[Android开发艺术探索-笔记02-IPC机制](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/)

本项目记录以下内容：
- AIDL在AS使用对常用技巧
- Android的Binder机制，Binder线程池的应用
- 进程间通信的各种知识

### 问题记录
**1. 引入aidl文件不自动生成对应对Java文件**
在gradle加入如下配置

```java
sourceSets {
    main {
        aidl.srcDirs = ['src/main/java']
    }
}
```

### 跨进程的几种方式
**1. 使用Bundle**

由于Bundle实现了Parcelable接口,所以在四大组件中的三大组件(Activity, Service, Receiver)都支持在Intent中传递Bundle.

**2. 使用文件共享**

文件共享适合在对数据同步要求不高的进程之间进行通信,并且要妥善的处理并发读写的问题.
场景：两个进程通过读/写同一个文件来交换数据. 例如进程A把数据写入文件中,而进程B从文件中读取出来数据.

**3. 使用Messenger**

Messenger(信使). 不同的进程中可以传递Message对象, 在Message中放入我们需要传递的数据,就可实现进程间传递. Messenger是一种轻量级的IPC方案,它的底层实现AIDL.
因为Messenger对AIDL进行了封装,使得在使用时更加简单,并且它的处理方式是一次处理一个请求,因此服务器端不用考虑线程同步因为服务端不存在并发执行的情形.

Messenger包含了一个有用的api-getBinder#==Retrieve the IBinder that this Messenger is using to communicate with its associated Handler.==

**4. 使用AIDL**

**5. 使用ContentProvider**

ContentProvider是Android提供专门用于不用应用进行数据共享的方式. 它的底层同样也是Binder. 因为系统封装, 所以它的使用比起AIDL要简单很多.

要实现一个内容提供者, 只需要写一个类继承ContentProvider,并复写六个抽象方法. 其中有四个是CURD操作方法. 一个onCreate()用来做初始化. 一个getType()用来返回一个Uri请求所对应的MIME类型,比如图片还是视频等. 如果我们不关心那么可是直接返回NULL或者*/*.
这六个方法根据Binder工作原理,都是运行在ContentProvider的进程中. 除了onCreate()是被系统回调运行在主线程, 其余的都在Binder的线程池中.

**6. 使用Socket**

Server:ServerSocket

**7. 各种IPC差异**

| 名称 | 优点 | 缺点 | 使用场景 |
|---|---|---|---|
| Bundle | 简单易用 | 只能传输Bundle支持的数据类型 | 四大组件间的进程间通信 |
| 文件共享 | 简单易用 | 不适合高并发场景,并且无法做到进程间的即时通 | 无法并发访问情形, 交换简单的数据实时性不高的场景 |
| AIDL | 功能强大 | 使用稍复杂,需要处理好线程同步 | 一对多通信且有RPC需求 |
| ContentProvider | 在数据源访问方面功能强大,支持一对多并发数据共享 | 可以理解为受约束的AIDL,主要提供数据源的CRUD操作 | 一对多的进程间的数据共享 |
| Messenger | 功能一般, 支持一对多串行通信,支持实时通信 | 不能很好处理高并发,不支持RPC,数据通过Message进行传输, 因此只能传输Bundle支持的数据类型 | 低并发的一对多即时通信,无RPC需求,或者无需要返回结果的RPC需求 |
| Socket | 功能强大,可以通过网络传输字节流,支持一对多并发实时通信 | 实现细节稍微有点繁琐,不支持直接的RPC | 网络数据交换 |

RPC（Remote Procedure Call Protocol）——远程过程调用协议

### Binder
日常开发中,Binder主要用在Service包括AIDL和Messenger. 而普通的Service中的Binder不涉及进程间的通信,无法触及Binder的核心. 而Messenger底层其实就是AIDL.所以我们利用AIDL来分析Binder的工作机制。
Binder的工作机制大体如下：

![](https://github.com/jacky1234/IPCDemo/blob/master/img/Binder_work.png?raw=true)

有两点需要注意:
- 客户端发起远程请求时,当前线程会被挂起直到服务器进程返回数据,所以注意线程是否在意耗时。
- 由于服务端Binder方法运行在Binder线程池中,所以不管Binder方法是否耗时都应该采用同步方式,因为已经在一个线程中了。
