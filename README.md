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

### Binder
日常开发中,Binder主要用在Service包括AIDL和Messenger. 而普通的Service中的Binder不涉及进程间的通信,无法触及Binder的核心. 而Messenger底层其实就是AIDL.所以我们利用AIDL来分析Binder的工作机制。
Binder的工作机制大体如下：
![](https://github.com/jacky1234/IPCDemo/img/Binder_work.png)
