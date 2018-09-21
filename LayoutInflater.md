# LayoutInflater

LayoutInflater主要是用来加载布局的，我们常看到的加载布局的任务通常都是在Activity中调用setContentView()方法来完成的，其实setContentView()方法的内部也是使用LayoutInflater来加载布局的。

LayoutInflater技术广泛应用于需要动态添加View的时候，比如在ScrollView和ListView中，经常都可以看到LayoutInflater的身影。 

## 基本用法

获取LayoutInflater的实例：

```java
LayoutInflater inflater1 = LayoutInflater.from(this); //后两种走的都是第一种
LayoutInflater inflater2 = getLayoutInflater();  
LayoutInflater inflater3 = (LayoutInflater) getSystemService(LAYOUT_INFLATER_SERVICE); 
```

得到了LayoutInflater的实例之后就可以调用它的inflate()方法来加载布局了：

```java
layoutInflater.inflate(resourceId, root); 
```

第一个参数就是要加载的布局id，第二个参数是指给该布局的外部再嵌套一层父布局，如果不需要就直接传null。

之后再将它添加到指定的位置就可以显示出来了。

`inflate(int resource, ViewGroup root, boolean attachToRoot)` 

如果root为null，attachToRoot将失去作用，设置任何值都没有意义。

如果root不为null，attachToRoot设为true，则会给加载的布局文件的指定一个父布局，即root。

如果root不为null，attachToRoot设为false，则会将布局文件最外层的所有layout属性进行设置，当该view被添加到父view当中时，这些layout属性会自动生效。

在不设置attachToRoot参数的情况下，如果root不为null，attachToRoot参数默认为true。

LayoutInflater是使用Android提供的pull解析方式来解析布局文件的，把整个布局文件都解析完成后就形成了一个完整的DOM结构，最终会把最顶层的根布局返回，至此inflate()过程全部结束。 

**注意：**

1.在`setContentView(R.layout.activity_main);` 方法中，Android会自动在布局文件的最外层再嵌套一个FrameLayout，所以activity_main中的layout_width和layout_height属性才会有效果，getParent()方法获取它的父布局 可以看到；

而对于普通的布局文件，它们的layout_width和layout_height属性就不会起作用，因为  layout_width和layout_height 其实是用于设置View在**布局中**的大小的 ，也就是说，首先View必须存在于一个布局中，但是布局文件本身已经是最外层了。

所以！LayoutInflater加载的布局文件最好套上一个parent layout,否则在view上设置的layout_width和layout_height将不起作用。

2.任何一个Activity中显示的界面其实主要都由两部分组成，标题栏和内容布局。

标题栏，就是在很多界面顶部显示的那部分内容，可以在代码中控制让它是否显示；

内容布局，就是一个FrameLayout，这个布局的id叫作content，我们调用setContentView()方法时所传入的布局其实就是放到这个FrameLayout中的，这也是为什么这个方法名叫作setContentView()，而不是叫setView()。 

