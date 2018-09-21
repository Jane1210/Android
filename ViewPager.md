## ViewPager

### 一 ViewPager的简单介绍

- ViewPager是android-support-v4.jar包中的一个系统控件
- ViewPager继承自ViewGroup
- ViewPager专门用以实现左右滑动切换View的效果

几个可以动态设置方法 ：

- `setAdapter(PagerAdapter adapter)` 设置适配器
- `setOffscreenPageLimit(int limit)` 设置缓存的页面个数,默认是 1
- `setCurrentItem(int item)` 跳转到特定的页面
- `setOnPageChangeListener(..)` 设置页面滑动时的监听器（现在API中建议使用 `addOnPageChangeListener(..)`）
- `setPageTransformer(..PageTransformer)` 设置页面切换时的动画效果
- `setPageMargin(int marginPixels)` 设置不同页面之间的间隔
- `setPageMarginDrawable(..)` 设置不同页面间隔之间的装饰图也就是 divide ，要想显示设置的图片，需要同时设置 `setPageMargin()`

### 二 PagerAdapter的使用

PagerAdapter类是与ViewPager配合使用的适配器类 

ViewPager将调用PagerAdapter来取得所需显示的页，而PagerAdapter也会在数据变化时，通知ViewPager。 

- **getCount()**

  获取ViewPager实际绘制的列表项的数量 

- destroyItem()

  当前项离开屏幕时回调本方法，在本方法中需要将当前项从ViewPager中移除 

- instantiateItem() 

  获取当前列表项，也就是正在显示的当前页，当前ViewPager所在的位置

- **isViewFromObject()**

  判断view和obj是否为同一个view 

### 三 缓存机制

ViewPager切换页面时默认情况下非相邻的页面会被销毁掉(ViewPager默认缓存相邻的页面以便快速切换）。

ViewPager中，常量DEFAULT_OFFSCREEN_PAGES = 1;如果想通过在setOffscreenPageLimit()方法中设置limit 为0,来禁用懒加载，是做不到的，默认会加载后一页；可以通过设置limit，来懒加载多页。

想要禁止懒加载，只能自定义viewPager来实现。
