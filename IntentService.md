## IntentService

#### 一 使用场景

IntentService相比父类Service而言，最大特点是其回调函数onHandleIntent中可以直接进行耗时操作，不必再开线程。其原理是IntentService的成员变量 Handler在初始化时已属于工作线程，之后handleMessage，包括onHandleIntent等函数都运行在工作线程中。 

但是IntentService还有一个特点，就是多次调用onHandleIntent函数（也就是有多个耗时任务要执行），多个耗时任务会按顺序依次执行。原理是其内置的Handler关联了任务队列，Handler通过looper取任务执行是顺序执行的。 

#### 二 IntentService与Service的区别

1.Service 

Service 是长期运行在后台的应用程序组件。

Service 不是一个单独的进程，它和应用程序在同一个进程中；

Service 也不是一个线程,它和线程没有任何关系，所以它不能直接处理耗时操作。

如果直接把耗时操作放在 Service 的 onStartCommand() 中，很容易引起 ANR（主线程阻塞） 。如果有耗时操作就必须开启一个单独的线程来处理。

2.IntentService

IntentService 是继承于 Service 并处理异步请求的一个类，在 IntentService 内有一个工作线程来处理耗时操作，启动 IntentService 的方式和启动传统 Service 一样，同时，当任务执行完后，IntentService 会自动停止，而不需要我们去手动控制。

另外，可以启动 IntentService 多次，而每一个耗时操作会以工作队列的方式在IntentService 的 onHandleIntent 回调方法中执行，并且，每次只会执行一个工作线程，执行完第一个再执行第二个，以此类推。

而且，所有请求都在一个单线程中，不会阻塞应用程序的主线程，同一时间只处理一个请求。

好处：首先，省去了在 Service 中手动开线程的麻烦；第二，当操作完成时，我们不用手动停止 Service。

#### 三 IntentService使用注意事项 

1.IntentService 默认实现了 OnBind()，返回值为 null。不适合用 `bindService()` 方法去启动。 

2.IntentService 必须实现 MyIntentService() 构造方法和 onHandleIntent(Intent intent)。

3.IntentService 的构造函数一定是参数为空的构造函数，然后再在其中调用 super(“name”) 这种形式的构造函数。 因为 Service 的实例化是系统来完成的，而且系统是用参数为空的构造函数来实例化Service的。 

