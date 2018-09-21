# WebView

### 一 简介

Android内置webkit内核的高性能浏览器，而WebView则是在这个基础上进行封装后的一个控件，WebView直译网页视图，我们可以简单的看作一个可以嵌套到界面上的一个浏览器控件。

#### 作用

- 显示和渲染Web页面
- 直接使用html文件（网络上或本地assets中）作布局
- 可和JavaScript交互调用 

WebView控件功能强大，除了具有一般View的属性和设置外，还可以对url请求、页面加载、渲染、页面交互进行强大的处理。 

### 二 webView常用方法

#### 1.加载URL

```java
//方式1. 加载一个网页：   
webView.loadUrl("http://www.google.com/");    
//方式2：加载apk包中的html页面   
webView.loadUrl("file:///android_asset/test.html");    
//方式3：加载手机本地的html页面    
webView.loadUrl("content://com.android.htmlfileprovider/sdcard/test.html");     
// 方式4： 加载 HTML 页面的一小段内容   
WebView.loadData(String data, String mimeType, String encoding)
```

####  2.WebView的状态

```java
//激活WebView为活跃状态，能正常执行网页的响应 
webView.onResume() ；  
//当页面被失去焦点被切换到后台不可见状态，需要执行onPause 
//通过onPause动作通知内核暂停所有的动作，比如DOM的解析、plugin的执行、JavaScript执行。  
webView.onPause()；  
//当应用程序(存在webview)被切换到后台时，这个方法不仅仅针对当前的webview而是全局的全应用程序的webview //它会暂停所有webview的layout，parsing，javascripttimer。降低CPU功耗。 
webView.pauseTimers() 
//恢复pauseTimers状态 
webView.resumeTimers()；  
//销毁Webview 
//在关闭了Activity时，如果Webview的音乐或视频，还在播放。就必须销毁Webview。 
//但是注意：webview调用destory时,webview仍绑定在Activity上，这是由于自定义webview构建时传入了该Activity的context对象，因此需要先从父容器中移除webview,然后再销毁webview: 
rootLayout.removeView(webView);  
webView.destroy();
```

####  3.关于前进 / 后退网页

```java
//是否可以后退 
Webview.canGoBack()  
//后退网页 
Webview.goBack()  
//是否可以前进                      
Webview.canGoForward() 
//前进网页 
Webview.goForward()  
//以当前的index为起始点前进或者后退到历史记录中指定的steps，如果steps为负数则为后退，正数则为前进 
Webview.goBackOrForward(intsteps) 
```

**常见用法：Back键控制网页后退**

- 问题：在不做任何处理前提下，浏览网页时点击系统的“Back”键,整个 Browser 会调用 finish()而结束自身
- 目标：点击返回后，是网页回退而不是退出浏览器
- 解决方案：在当前Activity中处理并销毁掉该 Back 事件

```java
public boolean onKeyDown(int keyCode, KeyEvent event) {     
    if (keyCode == KeyEvent.KEYCODE_BACK && mWebview.canGoBack()) {          
        mWebView.goBack();         
        return true;     
    }     
    return super.onKeyDown(keyCode, event); 
}
```

#### 4.清除缓存数据

```java
//清除网页访问留下的缓存 
//由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序. 
Webview.clearCache(true);  
//清除当前webview访问的历史记录 
//只会清除webview访问的历史记录，除了当前访问记录 
Webview.clearHistory()；  
//仅仅清除自动完成填充的表单数据，并不会清除WebView存储到本地的数据 
Webview.clearFormData()；
```

### 三 常用工具类

#### 1.WebSettings类

作用：对WebView进行配置和管理

**配置步骤：**

(1)添加访问网络权限

```xml
<uses-permission android:name="android.permission.INTERNET"/> 
```

(2)生成一个WebView组件（有两种方式） 

```java
//方式1：直接在在Activity中生成 
WebView webView = new WebView(this)  
//方法2：在Activity的layout文件里添加webview控件： 
WebView webview = findViewById(R.id.webView1);
```

(3)进行配置-利用WebSettings子类 

```java
//声明WebSettings子类 
WebSettings webSettings = webView.getSettings();  
//如果访问的页面中要与Javascript交互，则webview必须设置支持Javascript 
webSettings.setJavaScriptEnabled(true);   
//若加载的 html 里有JS 在执行动画等操作，会造成资源浪费（CPU、电量） 
//在 onStop 和 onResume 里分别把 setJavaScriptEnabled() 给设置成 false 和 true 即可  
//支持插件 
webSettings.setPluginsEnabled(true);   
//设置自适应屏幕，两者合用 
webSettings.setUseWideViewPort(true); //将图片调整到适合webview的大小  
webSettings.setLoadWithOverviewMode(true); // 缩放至屏幕的大小  
//缩放操作 
webSettings.setSupportZoom(true); //支持缩放，默认为true。是下面那个的前提。 
webSettings.setBuiltInZoomControls(true); //设置内置的缩放控件。若为false，则该WebView不可缩放 
webSettings.setDisplayZoomControls(false); //隐藏原生的缩放控件  
//其他细节操作 
webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); //关闭webview中缓存  
webSettings.setAllowFileAccess(true); //设置可以访问文件  
webSettings.setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口  
webSettings.setLoadsImagesAutomatically(true); //支持自动加载图片 
webSettings.setDefaultTextEncodingName("utf-8");//设置编码格式
```

**常见用法：设置WebView缓存**

- 当加载 html 页面时，WebView会在/data/data/包名目录下生成 database 与 cache 两个文件夹
- 请求的 URL记录保存在 WebViewCache.db，而 URL的内容是保存在 WebViewCache 文件夹下
- 是否启用缓存：

```java
//优先使用缓存: 
WebView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); 
//不使用缓存: 
WebView.getSettings().setCacheMode(WebSettings.LOAD_NO_CACHE);
//缓存模式如下：
//LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据
//LOAD_DEFAULT: （默认）根据cache-control决定是否从网络上取数据
//LOAD_NO_CACHE: 不使用缓存，只从网络获取数据
//LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据
```

- 结合使用（离线加载）

```java
if (NetStatusUtil.isConnected(getApplicationContext())) {     					
    webSettings.setCacheMode(WebSettings.LOAD_DEFAULT);//根据cache-control决定是否从网络上取数据
} else {     
    webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);//没网，则从本地获取，即离线加载
}  
webSettings.setDomStorageEnabled(true); // 开启 DOM storage API 功能
webSettings.setDatabaseEnabled(true);   //开启 database storage API 功能
webSettings.setAppCacheEnabled(true);	//开启 Application Caches 功能  
String cacheDirPath = getFilesDir().getAbsolutePath() + APP_CACAHE_DIRNAME; webSettings.setAppCachePath(cacheDirPath); //设置  Application Caches 缓存目录
```

**注意：** 每个 Application 只调用一次 WebSettings.setAppCachePath()，WebSettings.setAppCacheMaxSize() 

#### 2.WebViewClient类

作用：处理各种通知 & 请求事件

- **shouldOverrideUrlLoading()** 

当一个url即将被webview加载时，给Application一个机会来接管处理这个url，方法返回true代表Application自己处理url；返回false代表Webview处理url。   

打开网页时不调用系统浏览器， 而是在本WebView中显示；在网页上的所有加载都经过这个方法,这个函数我们 可以做很多操作。

```java
Webview webview = findViewById(R.id.webView1);
webView.loadUrl("http://www.google.com/");
//复写shouldOverrideUrlLoading()方法，使得打开网页时不调用系统浏览器， 而是在本WebView中显示
webView.setWebViewClient(new WebViewClient(){
   @Override
   public boolean shouldOverrideUrlLoading(WebView view, String url) {
      view.loadUrl(url);
      return true;
   }
});
```

- **onPageStarted()** 

  开始载入页面时调用，我们可以设定一个loading的页面，告诉用户程序在等待网络响应。

```java
webView.setWebViewClient(new WebViewClient(){      
    @Override      
    public void  onPageStarted(WebView view, String url, Bitmap favicon) {         
        //设定加载开始的操作      
    }  
});
```

- **onPageFinished()** 

  在页面加载结束时调用。我们可以关闭loading 条，切换程序动作。 

```java
webView.setWebViewClient(new WebViewClient(){       
    @Override       
    public void onPageFinished(WebView view, String url) {          
        //设定加载结束的操作       
    }   
});
```

- **onLoadResource()** 

  在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。 

```java
webView.setWebViewClient(new WebViewClient(){       
    @Override       
    public boolean onLoadResource(WebView view, String url) {          
        //设定加载资源的操作       
    }   
});
```

- **onReceivedError（）**  

  加载页面的服务器出现错误时（如404）调用。

```java
//步骤1：在assets文件夹下写一个html文件（error_handle.html），用于出错时展示给用户看的提示页面   
//步骤2：复写WebViewClient的onRecievedError方法，该方法传回了错误码，根据错误类型可以进行不同的错误分类处理     
webView.setWebViewClient(new WebViewClient(){       
    @Override       
    public void onReceivedError(WebView view, int errorCode, String description, String failingUrl){ 
        switch(errorCode){                 
            case HttpStatus.SC_NOT_FOUND:
                view.loadUrl("file:///android_assets/error_handle.html");
                break;                 
        }             
    }         
});
```

-  **onReceivedSslError()** 

  处理https请求。

```java
//webView默认是不处理https请求的，页面显示空白，需要进行如下设置： 
webView.setWebViewClient(new WebViewClient() {             
    @Override             
    public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
        handler.proceed();    //表示等待证书响应         
        // handler.cancel();      //表示挂起连接，为默认方式         
        // handler.handleMessage(null);    //可做其他处理         
    }         
});    
// 特别注意：5.1以上默认禁止了https和http混用，以下方式是开启 
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    mWebView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW); 
}
```

####  3.WebChromeClient类

作用：辅助 WebView 处理 Javascript 的对话框,网站图标,网站标题等等。

- **onProgressChanged（）** 

 获得网页的加载进度并显示 

```java
webview.setWebChromeClient(new WebChromeClient(){        
    @Override       
    public void onProgressChanged(WebView view, int newProgress) {           
        if (newProgress < 100) {               
            String progress = newProgress + "%";               
            progress.setText(progress);             
        } else {         
        }     
    });
```

-  **onReceivedTitle（）** 

 获取Web页中的标题 

```java
webview.setWebChromeClient(new WebChromeClient(){      
    @Override     
    public void onReceivedTitle(WebView view, String title) {        
        titleview.setText(title)；     
    }
```

-  **onJsAlert（）** 

支持javascript的警告框

一般情况下在 Android 中为 Toast，在文本里面加入\n就可以换行  

```java
webview.setWebChromeClient(new WebChromeClient() {                          
    @Override             
    public boolean onJsAlert(WebView view, String url, String message, final JsResult result)  {
        new AlertDialog.Builder(MainActivity.this)             
            .setTitle("JsAlert")             
            .setMessage(message)             
            .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                @Override                 
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm();                 
                }             
            })             
            .setCancelable(false)             
            .show();     
        return true;             
    }
```

-  **onJsConfirm（）** 

 支持javascript的确认框 

```java
webview.setWebChromeClient(new WebChromeClient() {                      
    @Override public boolean onJsConfirm(WebView view, String url, String message, final JsResult result) {     
        new AlertDialog.Builder(MainActivity.this)             
            .setTitle("JsConfirm")             
            .setMessage(message)             
            .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                @Override                 
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm();                 
                }             
            })             
            .setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
                @Override                 
                public void onClick(DialogInterface dialog, int which) {
                    result.cancel();                 
                }             
            })             
            .setCancelable(false)             
            .show(); 
        // 返回布尔值：判断点击时确认还是取消 
        // true表示点击了确认；false表示点击了取消；     
        return true; 
    }
```

-  **onJsPrompt（）** 

 支持javascript输入框

 点击确认返回输入框中的值，点击取消返回 null。 

```java
webview.setWebChromeClient(new WebChromeClient() {             
    @Override             
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, final JsPromptResult result) {     
        final EditText et = new EditText(MainActivity.this);     
        et.setText(defaultValue);     
        new AlertDialog.Builder(MainActivity.this)             
            .setTitle(message)             
            .setView(et)             
            .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                @Override                 
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm(et.getText().toString());                 
                }             
            })             
            .setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
                @Override                 
                public void onClick(DialogInterface dialog, int which) {
                    result.cancel();                 
                }             
            })             
            .setCancelable(false)             
            .show();      
        return true; 
    }
```

###  四 WebView滚动事件监听

#### 1.**使用场景**

当我们需要监听webview的滑动事件，来实现FloatingActionButton的显示和隐藏时；

当我们使用webview浏览html5页面的时候，希望可以记录当前浏览的位置，方便下次打开的时候，直接显示上次浏览到的位置时。

#### 2.**方法** 

 自定义一个继承WebView的类，重写onScrollChanged ()方法，并对外提供一个接口:

```java
public class ObservableWebView extends WebView {
    private OnScrollChangedCallback mOnScrollChangedCallback;
    @Override
    protected void onScrollChanged(final int l, final int t, final int oldl,
                                   final int oldt) {
        super.onScrollChanged(l, t, oldl, oldt);
 
        if (mOnScrollChangedCallback != null) {
            mOnScrollChangedCallback.onScroll(l - oldl, t - oldt);
        }
    }
    public static interface OnScrollChangedCallback {
        public void onScroll(int dx, int dy);//滚动时，x和y方向上的偏移量
    }
}
```

 监听webview的滚动事件，至少需要获得两个方面的信息：

(1) 滚动的偏移量

(2) 滚动的方向

###  五 Android通过WebView与JS交互

#### 1.Android调用JS

WebView调用JS有以下两种方式: 

- 通过WebView.loadUrl() 

- 通过WebView.evaluateJavascript()

方法2比方法1效率更高、使用更简洁，因为该方法的执行不会使页面刷新，而第一种方法的执行则会。不过

它要在Android 4.4 后才可使用。 

#### 2.JS调用Android

JS调用Native代码有以下三种方式： 

- 通过WebView.addJavascriptInterface()

- 通过WebViewClient.shouldOverrideUrlLoading()
- 通过WebChromeClient.onJsAlert()、onJsConfirm()、onJsPrompt()

WebView.addJavascriptInterface()是官方推荐的做法，在默认情况下WebView是关闭了JavaScript的调用，需要调用WebSetting.setJavaScriptEnabled(true)来进行开启。这个方法需要一个Object类型的JavaScript Interface，然后通过@JavascriptInterface来标注提供的JS调用的方法。

