## Handler

#### 一 Handler的概念

[Handler](http://developer.android.com/reference/android/os/Handler.html)，它直接继承自Object，一个Handler允许发送和处理Message或者Runnable对象，并且会关联到主线程的MessageQueue中。

每个Handler具有一个单独的线程，并且关联到一个消息队列的线程，就是说一个Handler有一个固有的消息队列。

当实例化一个Handler的时候，它就承载在一个线程和消息队列的线程，这个Handler可以把Message或Runnable压入到消息队列，并且从消息队列中取出Message或Runnable，进而操作它们。

Handler主要有两个作用：

- 在工作线程中发送消息。
- 在UI线程中获取、处理消息。

#### 二 Handler的执行流程图（异步消息处理机制）

![1533086914525](C:\Users\11084918\AppData\Local\Temp\1533086914525.png)

- **UI线程**:就是我们的主线程,系统在创建UI线程的时候会初始化一个Looper对象,同时也会创建一个与其关联的MessageQueue
- **Handler**:作用就是发送与处理信息,发送消息一般是使用Handler的sendMessage()方法，而发出的消息经过一系列辗转处理后，最终会传递到Handler的handleMessage()方法。如果希望Handler正常工作,在当前线程中要有一个Looper对象
- **Message**:Handler接收与处理的消息对象。它是在线程之间传递的消息，可以在内部携带少量的信息，用于在不同线程之间交换数据。
- **MessageQueue**:消息队列,先进先出管理Message,在初始化Looper对象时会创建一个与之关联的MessageQueue。它主要用于存放所有通过Handler发送的消息，这部分消息会一直存放在消息队列中，等待被处理。每个线程只会有一个MessageQueue对象。
- **Looper**:每个线程只能够有一个Looper,管理MessageQueue。调用Looper的loop()方法后，就会进入到一个无限循环中，每当发现MessageQueue中存在一条消息，就会将它取出，并传递到handleMessage()方法中。

异步消息处理的整个流程：

首先需要先在主线程中创建一个Handler对象,并重写handleMessage()方法；

```java
Handler handler = new Handler() {
    public void handlerMessage(Message msg) {
        switch(msg.what) {
            case UPDATE_TEXT:
                //UI操作
                text.setText("hhhhh");
                break;
            default:
                break;
        }
    }
};
```

当子线程中需要进行UI操作时，就创建一个Message对象，并通过Handler将这条消息发出去；

```java
new Thread(new Runnable() {
	@Override
    public void run() {
        Message msg = new Message();
        message.what = UPDATE_TEXT;
        handler.sendMessage(msg);
    }
}).start();
```

之后这条消息会被添加到MessageQueue的队列中等待被处理，而Looper则会一直尝试从MessageQueue中取出待处理的消息，最后根据message对象的what属性分发回Handler的handleMessage()方法中。

由于Handler是在主线程中创建的，所以此时handleMessage()方法中的代码也会在主线程中执行，便可以操作UI了。

#### 三 Handler的相关方法

- void **handleMessage**(Message msg):处理消息的方法,通常是用于被重写!
- **sendEmptyMessage**(int what):发送空消息
- **sendEmptyMessageDelayed**(int what,long delayMillis):指定延时多少毫秒后发送空信息
- **sendMessage**(Message msg):立即发送信息
- **sendMessageDelayed**(Message msg):指定延时多少毫秒后发送信息
- final boolean **hasMessage**(int what):检查消息队列中是否包含what属性为指定值的消息 如果是参数为(int what,Object object):除了判断what属性,还需要判断Object属性是否为指定对象的消息

#### 四 Handler的使用

##### 1.Handler写在主线程中

在主线程中,因为系统已经初始化了一个Looper对象,所以直接创建Handler对象,就可以进行信息的发送与处理了。

##### 2.Handler写在子线程中

如果是Handler写在了子线程中的话,我们就需要自己创建一个Looper对象了。创建的流程如下:

1）直接调用Looper.prepare()方法即可为当前线程创建Looper对象,而它的构造器会创建配套的MessageQueue; 

2）创建Handler对象,重写handleMessage( )方法就可以处理来自于其他线程的信息了

3）调用Looper.loop()方法启动Looper

#### 五 Handler与Message关系

Handler如果使用sendMessage的方式把消息入队到消息队列中，需要传递一个Message对象，而在Handler中，需要重写handleMessage()方法，用于获取工作线程传递过来的消息，此方法运行在UI线程上。

[Message](http://developer.android.com/reference/android/os/Message.html)是一个final类，所以不可被继承。

Message封装了线程中传递的消息，如果对于一般的数据，Message提供了getData()和setData()方法来获取与设置数据，其中操作的数据是一个[Bundle](http://developer.android.com/reference/android/os/Bundle.html)对象，这个Bundle对象提供一系列的getXxx()和setXxx()方法用于传递基本数据类型的键值对，对于基本数据类型，使用起来很简单。而对于复杂的数据类型，如一个对象的传递就要相对复杂一些。

在Bundle中提供了两个方法，专门用来传递对象的，但是这两个方法也有相应的限制，需要实现特定的接口，当然，一些Android自带的类，其实已经实现了这两个接口中的某一个，可以直接使用。方法如下：

- putParcelable(String key,Parcelable value)：需要传递的对象类实现Parcelable接口。
- putSerializable(String key,Serializable value)：需要传递的对象类实现Serializable接口。

还有另外一种方式在Message中传递对象，那就是使用Message自带的obj属性传值，它是一个Object类型，所以可以传递任意类型的对象，[Message](http://developer.android.com/reference/android/os/Message.html)自带的有如下几个属性：

- int arg1：参数一，用于传递不复杂的数据，复杂数据使用setData()传递。
- int arg2：参数二，用于传递不复杂的数据，复杂数据使用setData()传递。
- Object obj：传递一个任意的对象。
- int what：定义的消息码，一般用于设定消息的标志。

对于Message对象，一般并不推荐直接使用它的构造方法得到，而是建议通过使用Message.obtain()这个静态的方法或者Handler.obtainMessage()获取。Message.obtain()会从消息池中获取一个Message对象，如果消息池中是空的，才会使用构造方法实例化一个新Message，这样有利于消息资源的利用。并不需要担心消息池中的消息过多，它是有上限的，上限为10个。Handler.obtainMessage()具有多个重载方法，如果查看源码，会发现其实Handler.obtainMessage()在内部也是调用的Message.obtain()。　　
