# View

## 一 View与ViewGroup的概念及关系

Android的UI界面都是由View和ViewGroup及其派生类组合而成的。 

其中，View是所有UI组件的基类，Android中的任何一个布局、任何一个控件其实都是直接或间接继承自View的 ;而 ViewGroup是容纳这些组件的容器，其本身也是从View派生出来的。

View对象是Android平台中用户界面体现的基础单位。 

一般来说，开发Android应用程序的UI界面都不会直接使用View和ViewGroup，而是使用这两大基类的派生类。 

(1) View派生出的直接子类有：

AnalogClock,ImageView,KeyboardView, ProgressBar,SurfaceView, TextView,ViewGroup,ViewStub

(2) View派生出的间接子类有：

AbsListView,AbsSeekBar, AbsSpinner, AbsoluteLayout, AdapterView,AdapterViewAnimator, AdapterViewFlipper, AppWidgetHostView, AutoCompleteTextView,Button,CalendarView, CheckBox, CheckedTextView, Chronometer, CompoundButton  

(3)ViewGroup派生出的直接子类有：

AbsoluteLayout,AdapterView,FragmentBreadCrumbs,FrameLayout,LinearLayout,RelativeLayout,SlidingDrawer 

(4)ViewGroup派生出的间接子类有：

AbsListView,AbsSpinner, AdapterViewAnimator, AdapterViewFlipper, AppWidgetHostView, CalendarView, DatePicker, DialerFilter, ExpandableListView, Gallery, GestureOverlayView,GridView,HorizontalScrollView, ImageSwitcher,ListView 

## 二 View的绘制流程

每一个视图的绘制过程都必须经历三个最主要的阶段，即onMeasure()、onLayout()和onDraw()。

### 1.onMeasure()

**用于测量视图的大小。**

View系统的绘制流程会从ViewRoot的performTraversals()方法中开始，在其内部调用View的measure()方法。measure()方法接收两个参数：

- widthMeasureSpec，确定视图的宽度的规格和大小

- heightMeasureSpec，确定视图的高度的规格和大小 

MeasureSpec的值由specSize和specMode共同组成的，其中specSize记录的是大小，specMode记录的是规格。specMode一共有三种类型 ：

1. EXACTLY

   表示父视图希望子视图的大小应该是**由specSize的值来决定**的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。-MATCH_PARENT 

2. AT_MOST

   表示子视图**最多只能是specSize中指定的大小**，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。-WRAP_CONTENT 

3. UNSPECIFIED

   表示开发人员可以将视图按照自己的意愿设置成任意的大小，**没有任何限制**。这种情况比较少见，不太会用到。

一个界面的展示可能会涉及到很多次的measure，因为一个布局中一般都会包含多个子视图，每个视图都需要经历一次measure过程。

onMeasure()方法是可以重写的，如果你不想使用系统默认的测量方式，可以按照自己的意愿进行定制`setMeasuredDimension(200, 200);` 。View默认的测量流程依据就是布局文件中定义的MATCH_PARENT、WRAP_CONTENT等值。

总之，视图大小的控制是由父视图、布局文件、以及视图本身共同完成的，父视图会提供给子视图参考的大小，而开发人员可以在XML文件中指定视图的大小，然后视图本身会对最终的大小进行拍板。 

### 2.onLayout()

measure过程结束后，视图的大小就已经测量好了，接下来就是layout的过程了。 

这个方法是**用于给视图进行布局**的，也就是**确定视图的位置**。ViewRoot的performTraversals()方法会在measure结束后继续执行，并调用View的layout()方法来执行此过程： 

`host.layout(0, 0, host.mMeasuredWidth, host.mMeasuredHeight);` 

layout()方法接收四个参数，分别代表着左、上、右、下的坐标，当然这个坐标是相对于当前视图的父视图而言的。可以看到，这里还把刚才测量出的宽度和高度传到了layout()方法中。 

在layout()方法中，首先会调用setFrame()方法来判断视图的大小是否发生过变化，以确定有没有必要对当前的视图进行重绘，`boolean changed =  setFrame(l, t, r, b);` ，接下来调用onLayout()方法。

View中的onLayout()方法就是一个空方法，因为onLayout()过程是为了确定视图在布局中所在的位置，而这个操作应该是由布局来完成的，即父视图决定子视图的显示位置。

在onLayout()过程结束后，我们就可以调用getWidth()方法和getHeight()方法来获取视图的宽高了。 

【PS：getWidth()方法和getMeasureWidth()方法的区别（一般它们的值是相同的）：

首先getMeasureWidth()方法在measure()过程结束后就可以获取到了，而getWidth()方法要在layout()过程结束后才能获取到。另外，getMeasureWidth()方法中的值是通过setMeasuredDimension()方法来进行设置的，而getWidth()方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的。 】

### 3.onDraw()

measure和layout的过程都结束后，接下来就进入到draw的过程了。在这里才真正地开始对视图进行绘制。ViewRoot中的代码会继续执行并创建出一个Canvas对象，然后调用View的draw()方法来执行具体的绘制工作。 

关键步骤如下（本身总共有6步，但是2，5很少用到）：

**(1)对视图的背景进行绘制 。**

这里会先得到一个mBGDrawable对象，`Drawable background = mBGDrawable;` 然后根据layout过程确定的视图位置来设置背景的绘制区域，之后再调用Drawable的draw()方法来完成背景的绘制工作，`background.draw(canvas);` 。 这个mBGDrawable对象 其实就是在XML中通过android:background属性设置的图片或颜色。 

**(2)对视图的内容进行绘制。**

这里调用onDraw()方法，`onDraw(canvas);` 它是个空方法，因为每个视图的内容部分肯定都是各不相同的，这部分的功能交给子类来去实现也是理所当然的。  

**(3)对当前视图的所有子视图进行绘制。**

 `dispatchDraw(canvas);` View中的dispatchDraw()方法又是一个空方法，而ViewGroup的dispatchDraw()方法中就会有具体的绘制代码。 

**(4)对视图的滚动条进行绘制。** 

这是最后一步。`onDrawScrollBars(canvas);` 不止是ListView或者ScrollView ，不管是Button也好，TextView也好，任何一个视图都是有滚动条的，只是一般情况下我们都没有让它显示出来而已。 

## 三 Android视图状态及重绘流程分析

### 1.常用视图状态

**(1)enabled** 

表示当前视图是否可用。可以调用setEnable()方法来改变视图的可用状态，传入true表示可用，传入false表示不可用。它们之间最大的区别在于，不可用的视图是无法响应onTouch事件的。 

**(2)focused** 

表示当前视图是否获得到焦点。 一般只有视图在focusable和focusable in touch mode同时成立的情况下才能成功获取焦点，比如说EditText。 

**(3)window_focused** 

表示当前视图是否处于正在交互的窗口中，这个值由系统自动决定，应用程序不能进行改变。

**(4)selected**

表示当前视图是否处于选中状态。一个界面当中可以有多个视图处于选中状态，调用setSelected()方法能够改变视图的选中状态，传入true表示选中，传入false表示未选中。 

**(5)pressed**

表示当前视图是否处于按下状态。可以调用setPressed()方法来对这一状态进行改变，传入true表示按下，传入false表示未按下。通常情况下这个状态都是由系统自动赋值的，但开发者也可以自己调用这个方法来进行改变。 

比如，定义如下xml布局文件:

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/compose_pressed" android:state_pressed="true"></item>
    <item android:drawable="@drawable/compose_pressed" android:state_focused="true"></item>
    <item android:drawable="@drawable/compose_normal"></item>
</selector>
```

则，当视图处于正常状态的时候就显示compose_normal这张背景图，当视图获得到焦点或者被按下的时候就显示compose_pressed这张背景图。 

可以将它设置为某个按钮的背景图。

### 2.视图重绘

虽然视图会在Activity加载完成之后自动绘制到屏幕上，但是我们完全有理由在与Activity进行交互的时候要求动态更新视图，比如改变视图的状态、以及显示或隐藏某个控件等。那在这个时候，之前绘制出的视图其实就已经过期了，此时我们就应该对视图进行重绘。 

调用视图的setVisibility()、setEnabled()、setSelected()等方法时都会导致视图重绘，而如果我们想要手动地强制让视图进行重绘，可以调用invalidate()方法来实现。当然了，setVisibility()、setEnabled()、setSelected()等方法的内部其实也是通过调用invalidate()方法来实现的。

调用视图的invalidate()方法后确实会走到performTraversals()方法中，然后重新执行绘制流程 。

另外需要注意的是，invalidate()方法虽然最终会调用到performTraversals()方法中，但这时measure和layout流程是不会重新执行的，因为视图没有强制重新测量的标志位，而且大小也没有发生过变化，所以这时只有draw流程可以得到执行。而如果你希望视图的绘制流程可以完完整整地重新走一遍，就不能使用invalidate()方法，而应该调用requestLayout()了。 

## 四 事件分发与传递机制

### 1.事件分发的对象

点击事件（Touch事件） ，当用户触摸屏幕时（`View` 或 `ViewGroup`派生的控件），将产生点击事件（`Touch`事件） 

事件类型（4种） ：

| 事件类型                  | 具体动作                   |
| ------------------------- | -------------------------- |
| MotionEvent.ACTION_DOWN   | 按下View（所有事件的开始） |
| MotionEvent.ACTION_UP     | 抬起View（与DOWN对应）     |
| MotionEvent.ACTION_MOVE   | 滑动View                   |
| MotionEvent.ACTION_CANCEL | 结束事件（非人为原因）     |

即当一个点击事件（`MotionEvent` ）产生后，系统需把这个事件传递给一个具体的 `View` 去处理。 

### 2.事件在哪些对象之间进行传递

Activity、ViewGroup、View

### 3.事件分发的顺序

`Activity` -> `ViewGroup` -> `View` 

### 4.事件分发过程所需方法

dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()

## 五 Android自定义View的实现方法

按类型来划分的话，自定义View的实现方式大概可以分为三种，自绘控件、组合控件、以及继承控件。 

### 1.自绘控件

就是说，这个View上所展现的内容全部都是我们自己绘制出来的。绘制的代码是写在onDraw()方法中的 。

### 2.组合控件

就是说，我们并不需要自己去绘制视图上显示的内容，而只是用系统原生的控件就好了，但我们可以将几个系统原生的控件组合到一起，这样创建出的控件就被称为组合控件。 

举个例子来说，标题栏就是个很常见的组合控件，很多界面的头部都会放置一个标题栏，标题栏上会有个返回按钮和标题，点击按钮后就可以返回到上一个界面。 

### 3.继承控件

就是说，我们并不需要自己重头去实现一个控件，只需要去继承一个现有的控件，然后在这个控件上增加一些新的功能，就可以形成一个自定义的控件了。这种自定义控件的特点就是不仅能够按照我们的需求加入相应的功能，还可以保留原生控件的所有功能， 

