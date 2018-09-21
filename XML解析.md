## XML解析

### 一 xml语法

#### 1.定义

XML，即 `extensible Markup Language` ，是一种**数据标记语言** & **传输格式** 

区别于 `html` ：`html`用于显示信息；而 `XML`用于存储&传输信息 

#### 2.作用

对数据进行标记（结构化数据）、存储 & 传输 

可以把它看作是一个微型数据库，比如SharedPreferences就是使用xml文件来配置信息的，SQLite底层也是个xml文件；在网络应用方面，它通常作为信息的载体，我们通常把数据包装成xml来传递。

#### 3.语法

- 元素要关闭标签 

- 对大小写敏感

- 必须要有根元素(父元素）

- 属性值必须加引号

- 元素、属性、属性值、文本内容

- 元素与属性的差别：

  属性即提供元素额外的信息，但不属于数据组成部分的信息。

  ```xml
  <bookstore>
    <book category="CHILDREN">
       <title lang="en"> Harry Potter </title>
       <author> JK.Rowling</author>
    </book> 
  </bookstore>    
  ```

  ```xml
  <bookstore>   
      <book>      
          <category>CHILDREN<category>      
          <title lang="en"> Harry Potter </title>      
          <author> JK.Rowling</author>   
      </book>
  </bookstore>
  ```

  一般情况下，请使用元素(示例2)，因为

  - 属性无法描述树结构（元素可以）

  - 属性不容易拓展（元素可以）

  使用属性的情况：用于分配ID索引，用于标识XML元素。

  ```xml
  <bookstore>   
      <book id = "501">      
          <category>CHILDREN<category>      
          <title lang="en"> Harry Potter </title>      
          <author> JK.Rowling</author>   
      </book>
  </bookstore>            
  ```

  上述属性（id）仅用于标识不同的便签，并不是数据的组成部分。

### 二 xml树结构

XML文档中的元素会形成一种树结构，从根部开始，然后拓展到每个树叶（节点）。

#### 1.XML节点解释 

XML文件是由节点构成的。它的第一个节点为“根节点”。一个XML文件必须有且只能有一个根节点，其他节点都必须是它的子节点。 

this 代表整个XML文件，它的根节点就是 this.firstChild 。 this.firstChild.childNodes 则返回由根节点的所有子节点组成的节点数组。

每个子节点又可以有自己的子节点。节点编号由0开始，根节点的第一个子节点为 this.firstChild.childNodes[0]，它的子节点数组就是this.firstChild.childNodes[0].childNodes 。  

根节点第一个子节点的第二个子节点 this.firstChild.childNodes[0].childNodes[1]，它返回的是一个XML对象（Object） 。这里需要特别注意，节点标签之间的数据本身也视为一个节点 this.firstChild.childNodes[0].childNodes[1].firstChild ，而不是一个值。

- **节点**

名称：this.firstChild.childNodes[0].childNodes[1] 

文本内容：this.firstChild.childNodes[0].childNodes[1].firstChild 

- **值** 

名称：this.firstChild.childNodes[0].childNodes[1].nodeValue （节点名称有时也是我们需要的数据） 

文本内容：this.firstChild.childNodes[0].childNodes[1].nodeName 

### 三 常用的解析方式

主要分为两类：

#### 1.基于文档驱动

即在解析xml文档前，需先将整个xml文档存到内存中。

##### DOM方式

`Document Object Model`，即 文件对象模型，是 **一种 基于树形结构节点 & 文档驱动 的XML解析方法**。

定义了访问 & 操作xml文档元素的方法和接口。

应用场景：xml文档较小，需频繁操作，解析文档并且需要多次访问文档的情况。

```java
public void domParse(String filePath) {
    try {
        InputStream inputStream = getAssets().open(filePath);
        //生成一个文档工厂类
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        DocumentBuilder builder = factory.newDocumentBuilder();
        //开始解析输入流，因为dom解析是首先将整个文档读入内存，所以这里就完成了解析
        //下面是对这个文档取出操作
        Document document = builder.parse(inputStream);
        //获取到文档的根节点 manifest
        Element root = document.getDocumentElement();
        StringBuilder result = new StringBuilder();
        result.append("package:\n" + root.getAttribute("package") + "\n");
        //获取根节点的下一级节点的集合 application
        NodeList applications = root.getElementsByTagName("application");
        //开始解析<application>节点及其子节点
        for (int i = 0; i < applications.getLength(); i++) {
            Element application = (Element)applications.item(i);
            NodeList activities = application.getElementsByTagName("activity");
            if (activities.getLength() != 0)
                result.append("activities:\n");
            for (int j = 0; j < activities.getLength(); j++) {
                Element activity = (Element)activities.item(i);
                result.append(activity.getAttribute("android:name") + "\n");
            }
        }
        inputStream.close();
        text.setText(result.toString());
    } catch (ParserConfigurationException | IOException | SAXException e) {
        e.printStackTrace();
    }
}
```

#### 2.基于事件驱动

即根据不同需求事件（检索、修改、删除等）去执行不同解析操作，不需要将整个xml文档存储到内存中。SAX方式、PULL方式。

##### PULL方式

一种 基于事件流驱动 的`XML`解析方法。它的优点在于便捷高效。 

XmlPullParser.END_DOCUMENT 	XML文档的结束 

XmlPullParser.START_DOCUMENT 	XML文档的开始 

XmlPullParser.START_TAG 			标签的开始 

XmlPullParser.END_TAG 			标签的结束 

XmlPullParser.TEXT  				内容

```java
public void pullParse(String filePath) {
    InputStream inputStream = null;
    try {
        XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
        XmlPullParser pullParser = factory.newPullParser();
        inputStream = getAssets().open(filePath);
        pullParser.setInput(inputStream, "UTF-8");
        StringBuilder result = new StringBuilder();
        int eventType = pullParser.getEventType();
        //当还没有解析到结束文档的节点，一直循环
        while (eventType != XmlPullParser.END_DOCUMENT) {
            String nodeName = pullParser.getName();
            switch (eventType) {
                case XmlPullParser.START_DOCUMENT: //开始解析文档
                    break;
                case XmlPullParser.START_TAG:   //开始解析到节点，需要对每一个定义的节点进行处理
                    if ("manifest".equals(nodeName)) {
                        String packageName = pullParser.getAttributeValue(1);
                        result.append("package: " + packageName + "\n");
                    }else if ("activity".equals(nodeName)) {
                        String name = pullParser.getAttributeValue(0);
                        result.append("android:name: " + name + "\n");
                    }
                    break;
                case XmlPullParser.END_TAG: //结束节点，可以在这里将解析的Bean加入到集合中
                    break;
                case XmlPullParser.END_DOCUMENT:
                    break;
            }
            //获取到下一个节点，再触发解析动作
            eventType = pullParser.next();
        }
        text.setText(result.toString());
    } catch (XmlPullParserException | IOException e) {
        e.printStackTrace();
    } finally {
        if (inputStream != null) {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}  
```

##### SAX方式

即 `Simple API for XML`，**一种 基于事件流驱动、通过接口方法解析 的XML解析方法**。

与pull类似，需要写一个继承自DefaultHandler的`Handler`处理类并复写对应方法。
