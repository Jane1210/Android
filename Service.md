## Service

#### 一 服务是什么

服务是Android中实现程序后台运行的解决方案，它非常适合去执行那些不需要和用户交互而且还要求长期运行的任务。服务的运行不依赖于任何用户界面，即使被切换到后台，或用户打开另一个应用程序，服务仍然能够保持正常运行。

服务并不是运行在一个独立的进程中，而是依赖于创建服务时所在的应用程序进程。当该进程被杀掉，该服务也会停止运行。

#### 二 启动和停止服务

在服务的每个方法中打印日志，来验证是否成功启动或停止服务。

```java
Intent startIntent = new Intent(this,MyService.class);
startService(startIntent);//启动服务
stopService(stopIntent);//停止服务
```

#### 三 活动和服务进行通信

当一个活动和服务绑定后，就可以调用服务里面的Binder提供的方法了。

```java
private MyService.DownloadBinder downloadBinder;//在活动中根据场景调用DownloadBinder中的方法，用来实现指挥服务干什么
private ServiceConnection connection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
        downloadBinder = (MyService.DownloadBinder) iBinder;//向下转型
        downloadBinder.startDownload();
        downloadBinder.getProgress();
    }

    @Override
    public void onServiceDisconnected(ComponentName componentName) {
    }
};
```

```java
Intent bindIntent = new Intent(this, MyService.class);
bindService(bindIntent, connection, BIND_AUTO_CREATE);//绑定服务，第三个是标志位，BIND_AUTO_CREATE表示绑定后自动创建服务，则onCreat()方法执行，onStartCommand()不执行

unbindService(connection);//解绑服务
```

#### 四 两种启动方式比较

1.startService()启动Service

这样的Service与它的调用者无必然的联系，当调用者结束了自己的生命周期, 但是只要不调用stopService,那么Service还是会继续运行的。

无论启动了多少次Service,只需调用一次StopService即可停掉Service 。

2.bindService启动Service

bindService模式下的Service是与调用者相互关联的,可以理解为 "一条绳子上的蚂蚱",要死一起死,在bindService后,一旦调用者销毁,那么Service也立即终止 。

#### 五 服务的生命周期

- 调用Context的**startService()**方法：

		启动服务，回调onStartCommand()方法。若还未创建，会先调用onCreate()方法。

		启动后，服务会一直保持运行状态，直到活动中的stopService()或服务中的stopSelf()被调用。

- 调用Context的**stopService()**方法：

		回调onDestroy()方法。

- 调用Context的**bindService()**方法：

		获取一个服务的持久连接，回调服务中的onBind()方法。若还未创建，会先调用onCreate()方法。

		之后调用方获取到onBind()方法中返回的IBinder对象的实例，从而自由地和服务通信。只要连接没断开，就 
	
		一直保持运行状态。

- 调用Context的**stopService()**方法：

		回调onDestroy()方法。

- 同时调用**startService()和bindService()**方法：

		必须同时调用stopService()和stopService()，onDestroy()方法才会执行，服务才能被销毁。

![1533267300094](C:\Users\11084918\AppData\Local\Temp\1533267300094.png)
