## Android 进程间通信
参考自 《Android开发艺术探索》，eclipse下突然不能执行源代码`refusing to generate code from aidl file defining parcelable	Book.aidl`，苦于国内对VPN打压，暂时不能访问Google，所以决定将项目移至AS中。

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