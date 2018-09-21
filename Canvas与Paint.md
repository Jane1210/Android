# Canvas与Paint

Drawable以及Bitmap，都是加载好图片的，而Canvas(画布)，Paint(画笔)，Path(路径)是绘图相关的API，是我们 自定义View的基础。 

## 一 Paint(画笔)

画笔，该类保存了绘制几何图形、文本和位图的样式和颜色信息。也就是说我们可以使用Paint保存的样式和颜色，来绘制图形、文本和bitmap，这就是Paint的强大之处。 

```java
//直接使用无参构造方法就可以创建Paint实例
Paint mPaint = new Paint( ); 
//设置笔（Paint）的颜色和alpha值
mPaint.setColor(Color.BLUE);
mPaint.setAlpha(255);
```

### 1.负责设置获取图形绘制、路径相关的常用方法 

**setAlpha (int a)** 

该方法用于设置画笔的透明度，直观上表现为颜色变淡，具有一定的透明效果。该方法经常用于一些图片重叠或者特效显示的场合。 参数a为透明度，取值范围为0~255，数值越小越透明。 

**setStyle()** 

设置样式。总共有三种画笔的样式：

FILL					填充内容

STROKE				描边

FILL_AND_STROKE	填充内容并描边

**setStrokeWidth()**

设置画笔的宽度 

**setStrokeCap** 

设置线帽（相当于线多加的两头，圆角、直角，）。总共有三种线帽:

BUTT				没有线帽，默认模式

ROUND				圆形

SQUARE				方形

**setStrokeJoin()**

设置Join（线条连接处的样子），Join有三种类型： 

BEVEL：直线

ROUND：圆角

MITER：锐角

**setAntiAlias()** 

设置防锯齿，如果不设置，加载位图的时候可能会出现锯齿状的边界，如果设置，边界就会变的稍微有点模糊，锯齿就看不到了。 如果设置防锯齿，会损失一定的性能。

**setDither() **

设置是否抖动，如果不设置感觉就会有一些僵硬的线条，如果设置会使绘制的图片等颜色更加的清晰以及饱满，也是损失性能。  

**reset()** 

重置Paint，重新画

**setXfermode(Xfermode xfermode)** 

设置图形重叠时的处理方式，如合并，取交集或并集，经常用来制作橡皮的擦除效果

Xfermode 有三个子类：AvoidXfermode, PixelXorXfermode和PorterDuffXfermode，前两个都已过时，PorterDuffXfermode是唯一没过时的，就是图形混合模式的意思。 

**setMaskFilter(MaskFilter maskfilter)** 

MaskFilter类中没有任何实现方法，它的两个子类BlurMaskFilter和EmbossMaskFilter，前者为模糊遮罩滤镜而后者为浮雕遮罩滤镜。  

**setColorFilter(ColorFilter filter)**

可以直接传入colorFilter的子类ColorMatrixColorFilter、LightingColorFilter或PorterDuffColorFilter的子类对象作为参数。

### 2.负责设置获取文字相关的常用方法

**setTypeface(Typeface typeface)** 

设置字体样式，可以是Typeface设置的样式，也可以通过Typeface的createFromAsset(AssetManager mgr, String path)方法加载样式

**setShadowLayer(float radius, float dx, float dy, int shadowColor)** 

设置阴影效果，radius为阴影角度，dx和dy为阴影在x轴和y轴上的距离，color为阴影的颜色 

**setTextSize(float textSize)** 

设置字体大小 

**setTextSkewX(float skewX)** 

设置文本在水平方向上的倾斜，默认值为0，推荐的值为-0.25

**setLetterSpacing(float letterSpacing)** 

设置行的间距，默认值是0，负值行间距会收缩 

**breakText** 

剪切显示，即大于maxWidth的时候只截取指定长度的显示 

**mPaint.setTextAlign**(Align.LEFT) 

文本对齐方式 

## 二 Canvas(画布)

### 1.Canvas的含义

涉及到`Canvas`的主要有以下三个概念：画板、画布和图层，画板包含了多个画布，每个画布中间又可能会包含多个图层，同一层级的元素会叠加在一块，在上一层中间进行展示。

- **画板**
   画板用来承载画布。

- **画布**
   每一个画布就是一个`bitmap`，所有的图像都是画在`bitmap`上的。
   画布有两种：

  -  `View`的原始画布，也就是`onDraw(Canvas canvas)`当中传入的`canvas`实例。

  - 人造画布，即通过`saveLayer`或者`new Canvas(Bitmap)`这些方法来人为新建一个画布，`saveLayer`新建一个画布之后，以后所有的`draw`函数所画的图像都是在这个画布上的，只有在`restore、restoreToCount`函数以后，才会返回到原始画布上绘制。

- **图层**

  图层是绘制的最小单元，每次调用`canvas.drawXXX`函数，都会生成一个专门的透明图层来画这个图形，画完之后，就盖在了画布上。连续调用`draw`方法，那么就会生成多个透明图层，画完之后依次在**当前的画布**上显示。

### 2.获得Canvas

 (1)重写`View`中的函数 `onDraw(Canvas canvas)` `dispatchDraw(Canvas canvas)`

​     在`ViewGroup`中，当它有背景的时候调用`onDraw`方法，否则会跳过`onDraw`直接调用`dispatchDraw`方法，在  `View`中，两个都会调用的。 

(2)通过`Bitmap`创建

```java
Canvas c = new Canvas(bitmap);
//或者
Canvas c = new Canvas();
c.setBitmap(bitmap);
```

 如果我们通过这个方法构造了一个`canvas`，那么这个`canvas`上绘制的图像会保存在这个`bitmap`上，而不是在`View`上，如果想画在`View`上，就必须通过`onDraw`函数传进来的`canvas`画一遍。 

(3)`SurfaceHolder.lockCanvas()`

### 3.Canvas变换

可逆变换：

(1)平移

`void translate(float dx, dy)`，向右和向下为正方向，此时移动的是`canvas`原点的位置。 

(2)旋转

`rotate(float degrees, float px, float py)`，正数表示顺时针旋转，此外还可以指定旋转的中心，默认是`(0, 0)` 

(3)缩放

`scale(float sx, float sy, float px, py)`，对`x`和`y`进行缩放，并且可以指定缩放的中心点。 

(4)倾斜

`skew(float sx, float sy)`，倾斜画布，分别表示倾斜角度的`tan`值。 

 不可逆变换：

调用`clipXXX`函数进行裁剪，通过与`Rect、Path、Region`取交、并、差等集合运算来获得最新的画布状态，这个操作是不可逆的，一旦`canvas`被裁剪，那么就不能被恢复。 

###  4.Canvas绘制

#### (1)绘制基本图形

**线**

`drawLine(float startX, float startY, float stopX, float stopY, Paint paint)` 

**点**

`drawPoint(float x, float y, Paint paint)` 

**矩形**

`drawRect(float left, float top, float right, float bottom, Paint)` 

`drawRect(RectF rect, Paint paint)` 

**圆角矩形**

`drawRoundRect(RectF rect, float rx, float ry, Paint paint)` 

**圆**

`drawCircle(float cx, float cy, float radius, Paint paint)`   中心坐标，半径

**椭圆**

`drawOval(RectF oval, Paint paint)`  

#### (2)调用Path绘制

在`canvas`中，绘制路径时使用`drawPath(Path path, Paint paint)`方法 

**直线路径**

调用`moveTo(float x, float y)`可以移动到某个绘制点，`lineTo(float x, float y)`连到直线的结束点，如果连续画了几条直线，但没有形成闭环，调用`close()`会将路径首尾连接起来。 

**矩形路径**

`addRect(RectF rect, Path.Direction dir)`和`drawRect(float l, float t, float r, float b, Path.Direction dir)`，用来绘制一条封闭的矩形路径，第2个参数决定了该路径的方向。 

**圆形路径**

`addCircle(float x, float y, float radius, Path.Direction dir)`

以此类推。

#### (3)绘制文字

**普通水平绘制**

`drawText(String text, float x, float y, Paint paint)`

**指定各个文字位置**

`drawPosText(String text, float[] pos, Paint)`

**沿路径绘制**

`drawTextOnPath(String text, Path path, float hOffset, float vOffset, Paint paint)`，其中`hOffset`表示与路径起始点的水平偏移量，`vOffset`表示与路径中心的水平偏移量。

## 三 理解PorterDuffXfermode

PorterDuffXfermode类是setXfermode 方法的核心，也是图层混合模式里的核武器，通过它再加上我们的想象力，就能解决图形图像绘制里的很多问题 ，该类有且只有一个含参的构造方法：`PorterDuffXfermode(PorterDuff.Mode mode)`  

总共有18种模式，每种模式都有它自己的alpha通道和颜色值的计算方式。

各个模式下图像的合成效果如下：

![1534487643834](C:\Users\11084918\AppData\Local\Temp\1534487643834.png)

 其中用到的方法：

```java

setXfermode(new PorterDuffXfermode(PorterDuff.Mode.valueOf(String mode)))
```

 设置图层混合模式。

```
drawBitmap(Bitmap bitmap, Rect src, RectF dst, Paint paint) 
```

第一个Rect 代表对图片的裁剪，也就是说你想绘制图片的哪一部分；第二个 Rect 代表的是要将bitmap 绘制在屏幕的什么地方。

`saveLayer(RectF bounds, Paint paint, int saveFlags)` 

saveLayer可以为canvas创建一个新的透明图层，在新的图层上绘制，并不会直接绘制到屏幕上，而会在restore之后，绘制到上一个图层或者屏幕上（如果没有上一个图层）。

为什么会需要一个新的图层，例如在处理xfermode的时候，原canvas上的图（包括背景）会影响src和dst的合成，这个时候，使用一个新的透明图层是一个很好的选择。又例如需要当前绘制的图形都带有一定的透明度，那么创建一个带有透明度的图层，也是一个方便的选择。 

saveFlags：代表了需要保存哪方面的内容，这里一共有6种取值。ALL_SAVE_FLAG表示保存全部。

```
onSizeChanged(int w, int h, int oldw, int oldh)
```

 onSizeChanged()方法是系统自动调用的。 w，h是view当前的宽和高;oldw ,oldh是改变之前的宽和高。 

```
view.invalidate()
```

触发onDraw和computeScroll()。

