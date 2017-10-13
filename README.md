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


### Binder
日常开发中,Binder主要用在Service包括AIDL和Messenger. 而普通的Service中的Binder不涉及进程间的通信,无法触及Binder的核心. 而Messenger底层其实就是AIDL.所以我们利用AIDL来分析Binder的工作机制。
Binder的工作机制大体如下：

![](https://github.com/jacky1234/IPCDemo/blob/master/img/Binder_work.png?raw=true)

有两点需要注意:
- 客户端发起远程请求时,当前线程会被挂起直到服务器进程返回数据,所以注意线程是否在意耗时。
- 由于服务端Binder方法运行在Binder线程池中,所以不管Binder方法是否耗时都应该采用同步方式,因为已经在一个线程中了。
