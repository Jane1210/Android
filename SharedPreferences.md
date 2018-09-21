# SharedPreferences

不同于文件的存储方式，SharedPreferences是使用键值对的方式来存储数据的。

### 一 SharedPreferences读写

#### 1.将数据存储到SharedPreferences中

首先需要获取到SharedPreferences对象。主要有3种方法：

- Context类中的getSharedPreferences()方法
- Activity类中的getPreferences()方法
- PreferenceManager类中的getDefaultSharedPreferences()方法

得到SharedPreferences对象后，开始向SharedPreferences文件中存储数据，分3步实现：

- 调用SharedPreferences对象的edit()方法来获取一个SharedPreferences.Editor对象；
- 向SharedPreferences.Editor对象中添加数据，putBoolean()、putString()以此类推；
- 调用apply()方法将添加的数据提交，从而完成数据存储操作。

SharedPreferences文件是使用XML格式来对数据进行管理的。

#### 2.从SharedPreferences中读取数据

SharedPreferences对象中，针对SharedPreferences.Editor中的每一种put方法都有对应的get方法，getBoolean()、getString()以此类推。get方法有两个参数，第一个代表键，第二个代表默认值。

### 二 SharedPreferences数据保存位置

Android平台给我们提供的SharedPreferences类，是一个轻量级的存储类，特别适合用于保存软件配置参数。使用SharedPreferences保存数据，其背后是用xml文件存放数据，文件存放在/data/data/<package name>/shared_prefs目录下。

### 三 SharedPreferences使用注意事项

多进程间通过SharedPreferences来共享数据是不安全的，这个问题只能通过多进程间其它的通信方式或者是在确保不会同时操作SharedPreferences数据的前提下使用SharedPreferences来解决。可以试试MODE_MULTI_PROCESS模式。按照正常的MODE_PRIVATE模式，从SharedPreferences读出来的数据永远都是第一次写入的数据。
