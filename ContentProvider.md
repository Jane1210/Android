## ContentProvider

#### 一 ContentProvider概念讲解

内容提供器（ContentProvider）主要用于在不同的应用程序之间实现数据共享的功能，它提供了一套完整的机制，允许一个程序访问另一个程序中的数据，同时还能保证被访数据的安全性。目前，使用ContentProvider是Android实现跨程序共享数据的标准方式。

不同于文件存储和SharedPreferences存储中的两种全局可读写操作模式，ContentProvider可以选择只对那一部分数据进行共享，从而保证我们程序中的隐私数据不会有泄露的风险。

**ContentProvider执行原理**

**1.ContentResolver**

在ContentProvider的使用过程中，需要借用ContentResolver来控制ContentProvider所暴露处理的接口，作为代理来间接操作ContentProvider以获取数据。

在 Context.java 的源码中如下抽象方法： 

```java
public abstract ContentResolver getContentResolver();
```

所以可以在所有继承Context的类中通过 getContentResovler() 方法获取ContentResolver。 

**2.继承ContentProvider**

创建一个自定义ContentProvider的方式是继承ContentProvider类并实现其六个抽象方法（增删改查）。 

ContentProvider的六个抽象方法的含义分别是：

- onCreate()：代表ContentProvider的创建，可以用来进行一些初始化操作
- getType(Uri uri)：用来返回一个Uri请求所对应的MIME类型
- insert(Uri uri, ContentValues values)：插入数据
- query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)：查询数据
- update(Uri uri, ContentValues values, String selection, String[] selectionArgs)：更新数据
- delete(Uri uri, String selection, String[] selectionArgs)：删除数据

之后，需要在AdnroidManifest.xml中对ContentProvider进行注册。

![1533631436560](C:\Users\11084918\AppData\Local\Temp\1533631436560.png)

**3.URI**

MyContentProvider中要实现的6个抽象方法中，除了 onCreate() 方法，其它5个抽象方法都包含了一个Uri（统一资源标识符）参数，通过这个对象可以来匹配对应的请求。
URI 代表了要操作的数据对象，主要包含了两部分信息：

- 需要操作的哪个ContentProvider
- 对ContentProvider中的哪些数据进行操作

一个URI由以下几个部分组成：

- Scheme：ContentProvider的Scheme被规定为“content://”
- Authority：用于唯一标识某个ContentProvider，外部调用者根据这个标识来指定要操作的ContentProvider
- Path：用来标识要操作的具体数据

如何确定一个Uri是要执行哪项操作呢？要用UriMatcher来帮助ContentProvider匹配Uri，它仅包含了两个方法：

- addURI(String authority, String path, int code)：用于向UriMatcher对象注册Uri。

  authtity是在AndroidManifest.xml中注册的ContentProvider的authority属性值；

  path表示一个路径，可以设置为通配符，#表示任意数字，*表示任意字符；

  两者组合成一个Uri，而code则代表该Uri对应的标识码

- match(Uri uri)：匹配传入的Uri。返回addURI()方法中传递的code参数，如果找不到匹配的标识码则返回-1

**4.ContentValues** 

ContentValues 是一种存储的机制，只能存储基本类型的数据，像string，int之类的，不能存储对象。

在往数据库中插入数据时，首先应该有一个ContentValues的对象：

ContentValues initialValues = new ContentValues();

initialValues.put(key,values);

SQLiteDataBase sdb ;

sdb.insert(database_name,null,initialValues);

插入成功就返回记录的id否则返回-1；

#### 二 运行时权限

##### Android权限机制详解

运行时权限，即，用户不需要在安装软件时一次性授权所有申请的权限而是可以在软件的使用过程中再对某一项权限申请进行授权。

Android将所有的权限归为两类：

一是普通权限，系统自动授权，在AndroidManifest中权限声明即可；

二是危险权限，比如获取手机联系人等，必须由用户手动点击授权，需要做运行时权限处理。

#### 三 读取手机联系人

```java
private void getContacts() {
    // 查询raw_contacts表获得联系人的id
    ContentResolver resolver = getContentResolver();
    Uri uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI;
    //查询联系人数据
    Cursor cursor= resolver.query(uri, null, null, null, null);
    while (cursor.moveToNext()) {
        //获取联系人姓名、手机号码
        String name = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
        String cNum = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
        Log.d(TAG, "姓名： " + name);
        Log.d(TAG, "手机号码： " + cNum);
    }
    cursor.close();
}
```

```xml
<uses-permission android:name="android.permission.READ_CONTACTS"/>
```

#### 四 SQLiteDataBase、SQLiteOpenHelper、ContentProvider的区别

1.SQLiteOpenHelper是将对数据库和表的创建、插入、更新、删除操作进行了简单的封装

2.SQLiteDataBase代表一个数据库（底层就是一个数据库文件），一旦应用程序获得了代表指定数据库的SQLiteDataBase对象，接下来就可以通过SQLiteDataBase对象来管理和、操作数据库

3.而ContentProvider可以说是一个对外的接口，除了可以实现对SQLiteOpenHelper的封装，还可以实现对文件操作、图片操作、对象操作等实现封装

#### 五 数据库升级

由于Android的数据库SQLite是自带的，故随着我们的应用App升级，相对应的数据库里面的内容发生改变时也要随之升级，升级的时候希望的是之前保存的数据不丢失的情况下对其进行升级。这时就要用到Andoird的SQLiteOpenHelper类中有一个onUpgrade方法，当前数据库版本变化时就触发该方法对数据库进行升级。 

```java
public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion){     
    switch(oldVersion){       
        case 1:         
            db.execSQL(CREATE_CATEGORY);       
        case 2:         
            db.execSQL("alter table Book add column category_id integer");       
        default;     
    }   
}
```

