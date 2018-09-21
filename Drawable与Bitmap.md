# Drawable与Bitmap

## 一 13种Drawable

### Drawable资源使用注意事项

- Drawable分为两种： 

  - 普通的图片资源：一般放到res/mipmap目录下，AS会自动分hdpi，xhdpi... 

  - XML形式的Drawable资源：一般放到res/drawable目录 下，比如最常见的按钮点击背景切换的Selctor 

- 在XML中，通过@mipmap或者@drawable设置Drawable即可 

  在Java代码中，通过Resource的getDrawable(R.mipmap.xxx)可以获得drawable资源  

  如果是为某个控件设置背景，比如ImageView，我们可以直接调用控件.getDrawale() 

- Android中drawable中的资源名称有约束，必须是：**[a-z0-9_.]**（即：只能是字母数字及和.），而且不能以数字开头，否则编译会报错。注意是**小写字母**！

### 1.ColorDrawable

最简单的一种Drawable，当我们将ColorDrawable绘制到Canvas(画布)上的时候, 会使用一种固定的颜色来填充Paint,然后在画布上绘制出一片单色区域。

#### (1) Java中定义ColorDrawable

```java
ColorDrawable drawable = new ColorDrawable(0xffff2200);  
txtShow.setBackground(drawable); 
```

#### (2)xml中定义ColorDrawable

```xml
<?xml version="1.0" encoding="utf-8"?>  
<color  
    xmlns:android="http://schemas.android.com/apk/res/android"  
    android:color="#FF0000"/>  
```

当然上面这些用法,其实用得不多,更多的时候我们是在**res/values目录下创建一个color.xml** 文件,然后把要用到的颜色值写到里面,需要的时候通过@color获得相应的值，比如： 

#### (3)建立一个color.xml文件

```java
<?xml version="1.0" encoding="utf-8"?>  
<resources>  
    <color name="material_grey_100">#fff5f5f5</color>
    <color name="material_grey_300">#ffe0e0e0</color>
</resources>
```

在xml文件中，通过@color/xxx获得对应的color值；在Java中 ，

```java
int mycolor = getResources().getColor(R.color.mycolor);    
btn.setBackgroundColor(mycolor);   
```

#### (4)使用系统定义好的color

```java
btn.setBackgroundColor(Color.BLUE);
```

  xml中使用: **android:background="@android:color/black"** 

### 2.NiewPatchDrawable

注意：

- 点9图不能放在mipmap目录下，而需要放在drawable目录下 
- AS中的.9图，必须要有黑线，不然编译都不会通过 

#### xml定义NinePatchDrawable

```xml
<?xml version="1.0" encoding="utf-8"?>  
<nine-patch  
    xmlns:android="http://schemas.android.com/apk/res/android"  
    android:src="@drawable/dule_pic"  
    android:dither="true"/>  //是否对位图进行抖动处理
```

### 3.ShapeDrawable

根元素：**<shape../>**

形状的Drawable,定义基本的几何图形,如(矩形,圆形,线条等)。

<**shape**> 		形状的设置

<**size**> 			形状的宽度、高度

<**gradient**> 		渐变

<**solid**> 			color：背景填充色,设置solid后会覆盖gradient设置的所有效果 ！

<**stroke**> 		边框

<**conner**> 		圆角

<**padding**> 		边距

### 4.GradientDrawable

一个具有渐变区域的Drawable，可以实现线性渐变,发散渐变和平铺渐变效果 核心节点：<**gradient**/> 

**type**:渐变类型,可选(**linear**,**radial**,**sweep**), 线性渐变(可设置渐变角度),发散渐变(中间向四周发散),平铺渐变

**gradientRadius**:只有radial和sweep类型的渐变才有效,radial必须设置,表示渐变效果的半径 

### 5.BitmapDrawable

根节点：<**bitmap**>

对Bitmap的一种封装,可以设置它包装的bitmap在BitmapDrawable区域中的绘制方式,有: 平铺填充,拉伸填或保持图片原始大小。

**src**			图片资源 

**tileMode**	指定图片平铺填充容器的模式,设置这个的话,gravity属性会被忽略,有以下可选值: 

​			disabled**(整个图案拉伸平铺),**clamp**(原图大小), **repeat**(平铺),**mirror**(镜像平铺，相邻两张图对称)

### 6.InsetDrawable

表示把一个Drawable嵌入到另外一个Drawable的内部，并且在内部留一些间距, 类似于Drawable的padding属性,但padding表示的是Drawable的内容与Drawable本身的边距，而InsetDrawable表示的是两个Drawable与容器之间的边距，当控件需要的背景比实际的边框小的时候,比较适合使用InsetDrawable。

**drawable**	引用的Drawable,如果为空,必须有一个Drawable类型的子节点

**insetLeft**,**insetRight**,**insetTop**,**insetBottm**		设置左右上下的边距

### 7.ClipDrawable

ClipDrawable是通过设置一个Drawable的当前显示比例来裁剪出另一张Drawable，你可以通过调节这个比例来控制裁剪的宽高，以及裁剪内容占整个容器的权重，通过ClipDrawable的setLevel()方法调节显示比例可以实现类似Progress进度条的效果。ClipDrawable的level值范围在[0,10000]，level的值越大裁剪的内容越少，如果level为10000时则完全显示。 

**clipOrietntion**	设置剪切的方向,可以设置水平和竖直2个方向

**gravity**			从哪个位置开始裁剪

**drawable**		引用的drawable资源,为空的话需要有一个Drawable类型的子节点

(1) 定义一个ClipDrawable的资源xml;

(2) 在activity_main主布局文件中设置一个ImageView,将src设置为clipDrawable! 记住是src哦,如果你写成了blackground的话可是会报空指针的哦;

(3) MainActivity.java通过setLevel设置截取区域大小. 

### 8.RotateDrawable

用来对Drawable进行旋转,也是通过setLevel来控制旋转的,最大值也是:10000 

**fromDegrees**	起始的角度,,对应最低的level值,默认为0

**toDegrees**		结束角度,对应最高的level值,默认360

**pivotX**			设置参照点的x坐标,取值为0~1,默认是50%,即0.5

**pivotY**			设置参照点的Y坐标,取值为0~1,默认是50%,即0.5 ps:如果出现旋转图片显示不完全的话可以修 	改上述两个值解决!

**drawable**		设置位图资源

**visible**			设置drawable是否可见

用法跟clipDrawable一样。

### 9.AnimationDrawable

用来实现Android中帧动画，就是把一系列的 Drawable，按照一定的顺序一帧帧地播放。

**oneshot**	设置是否循环播放,false为循环播放 

**duration**	帧间隔时间,通常我们会设置为300毫秒

我们获得AniamtionDrawable实例后，需要调用它的start()方法播放动画，另外要注意 在OnCreate()方法中调用的话，是没有任何效果的，因为View还没完成初始化，我们可以用简单的handler来延迟播放动画。

### 10.LayerDrawable

根节点：<**layer-list**> 

层图形对象，包含一个Drawable数组，然后按照数组对应的顺序来绘制他们，索引 值最大的Drawable会被绘制在最上层！虽然这些Drawable会有交叉或者重叠的区域，但他们位于不同的层，所以并不会相互影响。 

**drawable**		引用的位图资源,如果为空要有一个Drawable类型的子节点

**left**/**right**...		层相对于容器的上下左右边距

**id**				层的id

可以实现拖动条、层叠图片等效果。以多个<item>的形式来实现。

### 11.TransitionDrawable

根节点：<**transition**> 

LayerDrawable的一个子类，TransitionDrawable只管理两层的Drawable， 并且提供了透明度变化的动画，可以控制一层Drawable过渡到另一层Drawable的动画效果。 

属性和LayerDrawable差不多， 我们需要调用**startTransition**方法才能启动两层间的切换动画； 也可以调用**reverseTransition()**方法反过来播放。

### 12.LevelListDrawable

根节点 :<**level-list**>

用来管理一组Drawable，我们可以为里面的drawable设置不同的level， 当他们绘制的时候，会根据level属性值获取对应的drawable绘制到画布上，它并没有可以设置的属性，我们能做的只是设置每个<**item**> 的属性！ 

**item可供设置的属性如下**：

> - **drawable**:引用的位图资源,如果为空要有一个Drawable类型的子节点
> - **minlevel:level**对应的最小值
> - **maxlevel:level**对应的最大值

### 13.StateListDrawable

根元素：<selector> 

它可以根据View的状态的不同匹配展示不同的Drawable。比如点击时背景是红色，不点击时时白色，所以StateListDrawable长于点击事件的背景。 

子节点要关心的只有 selector里面的属性和item里面的状态。 

每一个item表示一个Drawable。 

## 二 BitMap（位图）

区分几个名词的概念：

- **Drawable**：通用的图形对象，用于装载常用格式的图像，既可以是PNG，JPG这样的图像， 也可以是13种Drawable类型的可视化对象！我们可以理解成一个用来放画的——**画框**
- **Bitmap(位图)**：我们可以把它看作一个**画架**，我们先把画放到上面，然后我们可以 进行一些处理，比如获取图像文件信息，做旋转切割，放大缩小等操作
- **Canvas(画布)**：如其名，**画布**，我们可以在上面作画(绘制)，你既可以用**Paint(画笔)**， 来画各种形状或者写字，又可以用**Path(路径)**来绘制多个点，然后连接成各种图形

上述的这些都是Android中的底层图形类：**android.graphics**给我们提供的接口。

### 1.Bitmap，BitmapFactory，BitmapFacotry.Options

Bitmap的构造方法是私有的，外面不能实例化，只能通过JNI实例化

BitmapFactory 是提供给我们用来创建Bitmap的接口类，包含了很多decodeXxx ()方法，用来创建Bitmap

每一种decodeXxx ()方法，都会有一个Options类型的参数，这是一个静态内部类:BitmapFacotry.Options，用来设置decode时的选项 

BitmapFactory.Options的作用： 

1) 防止内存溢出 

2) 节省内存开销 

3) 系统更流畅   

### 2.Bitmap常用方法

普通方法：

- public boolean **compress** (Bitmap.CompressFormat format, int quality, OutputStream stream) 将位图的压缩到指定的OutputStream，可以理解成将Bitmap保存到文件中！ **format**：格式，PNG，JPG等； **quality**：压缩质量，0-100，0表示最低画质压缩，100最大质量(PNG无损，会忽略品质设定) **stream**：输出流 返回值代表是否成功压缩到指定流
- void **recycle**()：回收位图占用的内存空间，把位图标记为Dead
- boolean **isRecycled**()：判断位图内存是否已释放
- int **getWidth**()：获取位图的宽度
- int **getHeight**()：获取位图的高度
- boolean **isMutable**()：图片是否可修改
- int **getScaledWidth**(Canvas canvas)：获取指定密度转换后的图像的宽度
- int **getScaledHeight**(Canvas canvas)：获取指定密度转换后的图像的高度

静态方法：

- Bitmap **createBitmap**(Bitmap src)：以src为原图生成不可变的新图像
- Bitmap **createScaledBitmap**(Bitmap src, int dstWidth,int dstHeight, boolean filter)：以src为原图，创建新的图像，指定新图像的高宽以及是否变。
- Bitmap **createBitmap**(int width, int height, Config config)：创建指定格式、大小的位图
- Bitmap **createBitmap**(Bitmap source, int x, int y, int width, int height)以source为原图，创建新的图片，指定起始坐标以及新图像的高宽。
- public static Bitmap **createBitmap**(Bitmap source, int x, int y, int width, int height, Matrix m, boolean filter)

**BitmapFactory.Option**可设置参数： 

- boolean **inJustDecodeBounds**——如果设置为true，不获取图片，不分配内存，但会返回图片的高宽度信息。
- int **inSampleSize**——图片缩放的倍数。如果设为4，则宽和高都为原来的1/4，则图是原来的1/16。
- int **outWidth**——获取图片的宽度值
- int **outHeight**——获取图片的高度值
- int **inDensity**——用于位图的像素压缩比
- int **inTargetDensity**——用于目标位图的像素压缩比（要生成的位图）
- boolean **inScaled**——设置为true时进行图片压缩，从inDensity到inTargetDensity。

### 3.获取Bitmap位图

**(1)** **BitmapDrawable方法** 

```java
BitmapDrawable bmpMeizi = new BitmapDrawable(getAssets().open("pic_meizi.jpg"));
Bitmap mBitmap = bmpMeizi.getBitmap();
img_bg.setImageBitmap(mBitmap);
```

**(2)** **BitmapFactory方法** 

都是静态方法，直接调，可以通过资源ID、路径、文件、数据流等方式来获取位图.

```java
//通过资源ID
private Bitmap getBitmapFromResource(Resources res, int resId) {
      return BitmapFactory.decodeResource(res, resId);
}

//文件
private Bitmap getBitmapFromFile(String pathName) {
      return BitmapFactory.decodeFile(pathName);
}

//字节数组
public Bitmap Bytes2Bimap(byte[] b) {
    if (b.length != 0) {
        return BitmapFactory.decodeByteArray(b, 0, b.length);
    } else {
        return null;
    }
}

//输入流
private Bitmap getBitmapFromStream(InputStream inputStream) {
      return BitmapFactory.decodeStream(inputStream);
}
```

## 三 Bitmap与Drawable相互转换

### 1.**Drawable转换成Bitmap** 

```java
Resources res = getResources();
Bitmap bmp = BitmapFactory.decodeResource(res, R.drawable.simple);
```

### **2.Bitmap转换成Drawable** 

```java
Drawable drawable = new BitmapDrawable(bmp); 
```

## 四 避免Bitmap引起的OOM技巧

### 1.什么是OOM

**Out Of Memory**(内存溢出)，我们都知道Android系统会为每个APP分配一个独立的工作空间， 或者说分配一个单独的Dalvik虚拟机，这样每个APP都可以独立运行而不相互影响，而Android对于每个 Dalvik虚拟机都会有一个最大内存限制，如果当前占用的内存加上我们申请的内存资源超过了这个限制 ，系统就会抛出OOM错误。 

```java
// 获得正常的最大内存标准
ActivityManager activityManager = (ActivityManager)context.getSystemService(Context.ACTIVITY_SERVICE);
Log.e("HEHE","最大内存：" + activityManager.getMemoryClass());
```

### 2.避免Bitmap引起的OOM技巧

#### (1)采用低内存占用量的编码方式

```java
Bitmap backBitMap = Bitmap.createBitmap(width,height, Bitmap.Config.ARGB_8888);
```

ALPHA_8 代表8位Alpha位图

ARGB_4444 代表16位ARGB位图

ARGB_8888 代表32位ARGB位图

RGB_565 代表16位RGB位图

位图位数越高代表其可以存储的颜色信息越多，当然图像也就越逼真。

对于BitmapFactory.Options这个类，我们可以设置下其中的inPreferredConfig属性， 默认是Bitmap.Config.ARGB_8888，我们可以修改成Bitmap.Config.ARGB_4444。 

Bitmap.Config ARGB_4444：每个像素占四位，即A=4，R=4，G=4，B=4，那么一个像素点占4+4+4+4=16位 Bitmap.Config ARGB_8888：每个像素占八位，即A=8，R=8，G=8，B=8，那么一个像素点占8+8+8+8=32位 

默认使用ARGB_8888，即一个像素占4个字节！ 

#### (2)图片压缩

同样是BitmapFactory.Options，我们通过**inSampleSize**设置缩放倍数，比如写2，即长宽变为原来的1/2，图片就是原来的1/4，如果不进行缩放的话设置为1即可，但是不能一味的压缩，毕竟这个值太小的话，图片会很模糊，而且要避免图片的拉伸变形，所以需要我们在程序中动态的计算inSampleSize的合适值。

Options中又有这样一个方法：**inJustDecodeBounds**，将该参数设置为 true后，decodeFiel并不会分配内存空间，但是可以计算出原始图片的长宽，调用 options.**outWidth**/**outHeight**获取出图片的宽高，然后通过一定的算法，即可得到适合的 inSampleSize 。

#### (3)及时回收图像

如果引用了大量的Bitmap对象，而应用又不需要同时显示所有图片。可以将暂时不用到的Bitmap对象 及时回收掉。 

Bitmap.recycle(); 释放图片的资源，此时Bitmap本身并没有释放，它依然在占用资源，所以还要再调用一次Bitmap=null; 将Bitmap赋空，让有向图断掉，好让GC回收。 
