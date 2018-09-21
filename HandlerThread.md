## HandlerThread

Java与Android在线程方面大部分是相同的，而HandlerThread是Aandroid的专属类。

HandlerThread本质上就是一个普通Thread,只不过内部建立了Looper（而除主线程外，Android中新诞生的线程是没有开启消息循环的 ). 

#### 一 HandlerThread的特点

- HandlerThread本质上是一个线程类，它继承了Thread；
- HandlerThread有自己的内部Looper对象，可以进行looper循环；
- 通过获取HandlerThread的looper对象传递给Handler对象，可以在handleMessage方法中执行异步任务。
- 创建HandlerThread后必须先调用HandlerThread.start()方法，Thread会先调用run方法，创建Looper对象。

#### 二 HandlerThread常规使用步骤

- 创建一个HandlerThread

```java
mThread = new HandlerThread("handler_thread");
```

- 启动一个HandlerThread

```java
mThread.start();
```

- 退出循环
  Looper是通过调用loop方法驱动着消息循环的进行: 从MessageQueue中阻塞式地取出一个消息，然后让Handler处理该消息，周而复始，loop方法是个死循环方法。

可以调用Looper的quit方法或quitSafely方法 来终止循环。

#### 三 handler与handlerThread的区别

1.Handler在android里负责发送和处理消息。

它的主要用途有：

- 按计划发送消息或执行某个Runnanble(使用POST方法)；

- 从其他线程中发送来的消息放入消息队列中，避免线程冲突（常见于更新UI线程） 

子线程是运行在UI主线程中的，不适合进行比较耗时的操作，会阻塞主线程UI队列。  

2.HandlerThread继承于Thread，所以它本质就是个Thread。  

与普通Thread的差别就在于，它有个Looper成员变量。这个Looper其实就是对消息队列以及队列处理逻辑的封装，简单说就是 消息队列+消息循环。  

通过HandlerThread新建的线程是一个新的线程，不是运行在UI主线程中的，不能操作UI，但是可以通过Message发送消息来进行操作。

可以用来处理一些比较耗时的操作。   
