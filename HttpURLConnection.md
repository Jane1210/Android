# HttpURLConnection

### 一 HttpURLConnection的介绍

一种多用途、轻量极的HTTP客户端，使用它来进行HTTP操作可以适用于大多数的应用程序。虽然HttpURLConnection的API提供的比较简单，但是同时这也使得我们可以更加容易地去使用和扩展它。继承自URLConnection，抽象类，无法直接实例化对象。通过调用**openCollection**() 方法获得对象实例，默认是带gzip压缩的。

### 二 HttpURLConnection的使用步骤 

- 创建一个URL对象： **URL url = new URL(http://www.baidu.com);**

- 调用URL对象的openConnection( )来获取HttpURLConnection对象实例： 

  **HttpURLConnection conn = (HttpURLConnection) url.openConnection();**

- 设置HTTP请求使用的方法:GET或者POST，或者其他请求方式比如：PUT **conn.setRequestMethod("GET");**

- 设置连接超时，读取超时的毫秒数，以及服务器希望得到的一些消息头 **conn.setConnectTimeout(6\*1000);** **conn.setReadTimeout(6 \* 1000);**

- 调用getInputStream()方法获得服务器返回的输入流，然后输入流进行读取

   **InputStream in = conn.getInputStream();**

- 最后调用disconnect()方法将HTTP连接关掉 **conn.disconnect();**

**PS**:除了上面这些外,有时我们还可能需要对**响应码**进行判断,比如200: if(conn.getResponseCode() != 200)，然后作一些处理。

### 三 HttpURLConnection的常用方法

**get请求常用方法：**

得到connection对象：
HttpURLConnection connection = (HttpURLConnection) url.openConnection();

设置连接超时：
conn.setConnectTimeout(5000);

设置请求方式：

connection.setRequestMethod("GET");

设置此类是否应该自动执行 HTTP 重定向：

setFollowRedirects() ;

建立到远程对象的实际连接 （请求行，请求头的设置必须放在网络连接前 ）：
connection.connect();
得到响应码：
int responseCode = connection.getResponseCode();

得到响应流（只是得到一个流对象，并不是数据，从流中读取数据的操作必须放在子线程 ）：
InputStream inputStream = connection.getInputStream();

如果需要传递参数，则直接把参数拼接到url后面，其他完全相同，如下：

```
String url = "https://www.baidu.com/?userName=zhangsan&password=123456";
```

**post请求常用方法**：

HttpURLConnection connection = (HttpURLConnection) url.openConnection();
connection.setRequestMethod("POST");//设置请求方式为POST
connection.setDoOutput(true);//允许写出
connection.setDoInput(true);//允许读入
connection.setUseCaches(false);//不使用缓存
 connection.connect();//连接

post方式传递参数的本质是：从连接中得到一个输出流，通过输出流把数据写到服务器。  

Get请求与post请求都可以设置请求头，设置请求头的方式也是相同的：

```java
connection.setRequestMethod("POST");
connection.setRequestProperty("version", "1.2.3");//设置请求头
connection.setRequestProperty("token", token);//设置请求头
connection.connect();
```

PS：一般我们实际开发并不会用HttpURLConnection和HttpClient，使用别人封装 好的第三方网络请求框架，诸如：Volley，android-async-http，loopj等，因为网络操作涉及到异步以及多线程，自己动手的话，很麻烦，所以实际开发还是直接用第三方。 
