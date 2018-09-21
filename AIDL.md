## AIDL

### 一 概述

AIDL (Android Interface Definition Language , Android接口定义语言 )，是用于定义服务器和客户端通信接口的一种描述语言，可以拿来生成用于IPC(Inter-Process Communication，进程间通信 )的代码。 

设计AIDL这门语言的目的就是为了实现进程间通信。 

通过AIDL，可以在一个进程中获取另一个进程的数据和调用其暴露出来的方法，从而满足进程间通信的需求。

通常，暴露方法给其他应用进行调用的应用称为服务端，调用其他应用的方法的应用称为客户端，客户端通过绑定服务端的Service来进行交互 。

### 二 语法

AIDL语法与Java基本一致，需要记住的规则有以下几点： 

1.AIDL文件以 **.aidl** 为后缀名

2.AIDL支持的数据类型：

- 8种基本类型
- String，CharSequence
- 实现了Parcelable接口的数据类型
- List类型，List承载的数据必须是AIDL支持的类型，或者是其它声明的AIDL对象 
- Map类型，List承载的数据必须是AIDL支持的类型，或者是其它声明的AIDL对象 

3.AIDL文件可以分为两类：

   a.用来声明实现了Parcelable接口的数据类型，以供其他AIDL文件使用那些非默认支持的数据类型 

   b.用来定义接口方法，声明要暴露哪些接口给客户端调用，定向Tag就是用来标注这些方法的参数值 

4.定向Tag:

   表示在跨进程通信中数据的流向，用于标注方法的参数值。

   in: 表示数据只能由客户端流向服务端 

   out: 表示数据只能由服务端流向客户端 

   inout: 表示数据可在服务端与客户端之间双向流通 

   如果AIDL方法接口的参数值类型是：基本数据类型、String、CharSequence或者其他AIDL文件定义的方法接口，那么这些参数值的定向 Tag 默认是且只能是 in;除了这些类型外，其他参数值都需要明确标注使用哪种定向Tag 。

5.明确导包：

   在AIDL文件中需要明确标明引用到的数据类型所在的包名，即使两个文件处在同个包名下。

### 三 服务端编码

注意：创建或修改过AIDL文件后需要clean下工程（Build下），使系统及时生成我们需要的文件。在generated目录下查看。 

1.aidl文件：

​	定义Parcelable数据类型

​	定义接口方法，声明要暴露哪些接口给客户端调用，tag标注方法参数（系统由此自动生成对应的.java文件，里面含有内部静态抽象类.Sub，在进程间通信中真正起作用 ）

2.java文件：

​	定义实体类；

​	定义Service，供远程客户端绑定，实现aidl文件中定义的接口方法

注意：因为服务端的Service需要被客户端来远程绑定，所以客户端要能够找到这个Service，可以通过先指定包名，再配置Action值或者直接指定Service类名的方式来绑定Service；
 如果是通过先指定包名，再配置Action值方式来绑定Service，需要在AndroidManifest.xml中配置action.

### 四 客户端编码

1.将Server端的aidl文件夹完全复制到客户端，与java文件夹同级目录；

2.创建和服务端实体类所在的相同包名来存放实体类；

3.设计客户端布局；

4.在UI界面给控件注册点击事件，绑定服务端。

这样，客户端和服务器之间的通信就实现了，客户端能获取到服务端的数据，也能向服务端传递数据。

注意：运行客户端之前要先让服务端处于运行状态。

### 五 注册回调函数

当服务端要进行耗时操作，然后才返回结果给客户端时，客户端可以向服务端注册一个回调函数用于接收运算结果，而不用一直等待返回值。

回调函数的序列化：

Binder 会把客户端传过来的对象序列化后转为一个新的对象传给服务端，即使客户端使用的一直是同个对象，但对服务端来说前后两个回调函数其实都是两个完全不相关的对象，对象的跨进程传输本质上都是序列化与反序列化的过程。

为了能够无误地注册和解除注册回调函数，系统为开发者提供了 `RemoteCallbackList`，RemoteCallbackList 是一个泛型类，系统专门提供用于删除跨进程回调函数，支持管理任意的 AIDL 接口，因为所有的 AIDL 接口都继承自 `IInterface`，而  RemoteCallbackList 对于泛型类型有限制：

```java
public class RemoteCallbackList<E extends IInterface>
```

 RemoteCallbackList 在内部有一个 ArrayMap 用于 保存所有的 AIDL 回调接口 

```java
ArrayMap<IBinder, Callback> mCallbacks  = new ArrayMap<IBinder, Callback>();
```

其中 Callback 封装了真正的远程回调函数，因为即使回调函数经过序列化和反序列化后会生成不同的对象，但这些对象的底层 Binder 对象是同一个。利用这个特征就可以通过遍历 RemoteCallbackList 的方式删除注册的回调函数了。

 此外，当客户端进程终止后，RemoteCallbackList 会自动移除客户端所注册的回调接口。而且 RemoteCallbackList 内部自动实现了线程同步的功能，所以我们使用它来注册和解注册时，不需要进行线程同步。

 注意：如果确定远程方法是耗时的，就要避免在 UI 线程中去调用远程方法。所以，客户端调用远程方法的操作可以放到子线程中进行。此外，客户端的 `ServiceConnection`对象的 `onServiceConnected` 和 `onServiceDisconnected`都是运行在 UI 线程中，所以也不能用于调用耗时的远程方法。 

而服务端就不同了，由于服务端的方法本身就运行在服务端的 Binder 线程池中，所以服务端方法本身就可以用于执行耗时方法，不必再在服务端方法中开线程去执行异步任务。 

 





​                 



   
