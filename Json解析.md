# Json解析

### 一 Json语法

#### 1.JSON是什么

1. JSON的全程是JavaScript Object Notation，也就是JavaScript 对象表示法
2. JSON是存储和交换文本信息的语法，类似XML，但是比XML更小、更快，更易解析
3. JSON是轻量级的文本数据交换格式，独立于语言，具有自我描述性，更易理解

#### 2.JSON语法是 JavaScript 对象表示法语法的子集

- 数据在 名称/值对 中
- 数据由逗号分隔
- 花括号{}保存对象
- 方括号[]保存数组

#### 3.JSON 名称/值对

```
"name" : "zhangsan"
"age" : 15
```

#### 4.JSON 值

JSON 值可以是：

- 数字（整数或浮点数）
- 字符串（在双引号中）
- 逻辑值（true 或 false）
- 数组（在方括号[]中）
- 对象（在花括号{}中）
- null

#### 5.JSON 对象

JSON 对象在花括号{}中书写： 对象可以包含多个名称/值对，比如： 

```
{"name":"zhangsan" , "age":25}
```

#### 6.JSON 数组

JSON 数组在方括号[]中书写： 数组可包含多个对象，比如： 

```
{"student":[{"name":"zhangsan" , "age":12},{"name":"lisi" , "age":13},{"name":"wangwu" , "age":14}]}
```

### 二 JSON解析方向

1、将Java对象（包括集合）转换成JSON格式字符串；（服务端）

2、将JSON格式字符串转换成Java对象（包括集合）。（客户端）

3、JSON和Java之间的转换关系：

​      JSON中的对象对应着Java中的对象；

​      JSON中的数组对应着Java中的集合。

### 三 JSON解析技术

#### 1.JsonReader

(1)使用JsonReader方法解析Json数据对象，你需要创建一个JsonReader对象

(2)然后使用beginArray()来开始解析 [ 左边的第一个数组

(3)再使用beginObject()来开始解析数组{中的第一个对象

(4)对于直接的数据可以直接得到解析到的数据，但对于在json中嵌套了数组的数据，需要在写一个解析方法

(5)在解析完成后，别忘用endArray(),endObject()来关闭解析

```java
private void parseJsonArrayByJsonReader(String jsons) {
    JsonReader jsonReader = new JsonReader(new StringReader(jsons));
    try {
        jsonReader.beginArray();
        while (jsonReader.hasNext()) {
            jsonReader.beginObject();
            while (jsonReader.hasNext()) {
                String tagName = jsonReader.nextName();
                if (tagName.equals("id")) {
                    id = jsonReader.nextInt();
                } else if (tagName.equals("name")) {
                    name = jsonReader.nextString();
                } //.......
            }
            jsonReader.endObject();
        }
        jsonReader.endArray();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

#### 2.Android原生技术

特点：很麻烦，对于复杂的json数据解析很容易出错（不推荐使用）

- **解析JSON对象的API：JsonObject** 

JSONObject(String json)；将Json字符串解析成Json对象；

XxxgetXxx(String name) ；根据name在json对象中得到相应的value。

(1)获取或创建JSON数据

如，自己创建一个JSON数据：

```java 
private String json = "{\n" +
        "\t\"id\":2, \"name\":\"苹果\", \n" +
        "\t\"price\":12.3, \n" +
        "}\n"; 
```

(2)解析Json对象（这里记得要加try_catch异常处理 ）

```java
try {
    JSONObject jsonObject = new JSONObject(json);
    int id = jsonObject.optInt("id");
    String name = jsonObject.optString("name");
    double price = jsonObject.optDouble("price");
    // 封装Java对象
    shopInfo = new ShopInfo(id, name, price);
} catch (JSONException e) {
    e.printStackTrace();
}
```

(3)显示解析后的数据 

- **解析Json数组的API：JSONArray** 

JSONArray(String json)；将json字符串解析成json数组；

int length()；得到json数组中元素的个数；

XxxgetXxx(int i) ；根据下标得到json数组中对应的元素数据。

(1)获取或创建JSON数据

```java
String json = "[\n" +
                "    {\n" +
                "        \"id\": 1,\n" +
                "        \"name\": \"金鱼1\",\n" +
                "        \"price\": 12.3\n" +
                "    },\n" +
                "    {\n" +
                "        \"id\": 2,\n" +
                "        \"name\": \"金鱼2\",\n" +
                "        \"price\": 12.5\n" +
                "    }\n" +
                "]";
```

 (2)解析json数组

```java
try {            
    JSONArray jsonArray = new JSONArray(json);             
    for (int i = 0; i < jsonArray.length(); i++) {                
        JSONObject jsonObject = jsonArray.getJSONObject(i);                 
        if (jsonObject != null) {                    
            int id = jsonObject.optInt("id");                     
            String name = jsonObject.optString("name");                     
            double price = jsonObject.optDouble("price");                                       
            // 封装Java对象                    
            ShopInfo shopInfo = new ShopInfo(id, name, price);                     			
            shops.add(shopInfo);                
        }            
    }        
} catch (JSONException e) {            
    e.printStackTrace();        
}  
```

(3)显示解析后的数据

至于复杂特殊的json数据，也是这样一层一层解析。

#### 3.Gson框架技术

特点：解析没那么麻烦，代码量简洁，可以很方便地解析复杂的Json数据，而且谷歌官方也推荐使用。 

(1)将gson的jar包导入到项目libs目录下，或者直接通过Gradle添加依赖

```java
implementation group: 'com.google.code.gson', name: 'gson', version: '2.8.0'
```

(2)创建Gson对象 

```
Gson gson = new Gson(); 
```

注意要记得创建对象的JavaBean类；要求json对象中的key的名称与Java对象的JavaBean类中的属性名要相同，否则解析不成功！ 

- **用Gson解析JSON对象**

通过创建的Gson对象调用fromJson()方法，返回该json数据对象的Java对象;显示数据

```
shopInfo = gson.fromJson(json, ShopInfo.class); 
```

- **用Gson解析JSON数组**

```java
private void parseJSONArrayByGson(String jsons) {
    Gson gson = new Gson();
    //返回该json数据对应的Java集合
    shops = gson.fromJson(jsons, new TypeToken<List<ShopInfo>>(){}.getType());
}
```

#### 4.FastJson框架技术

特点：Fastjson是用Java语言编写的高性能功能完善的JSON库。它采用了一种“假定有序、快速匹配”的算法，

把JSON Parse的性能提升到极致，是目前Java语言中最快的JSON库。速度大约是Gson的6倍。

(1)要使用Fastjson，也是想Gson一样，先导入jar包，或者在Gradle中添加依赖

```java
implementation 'com.alibaba:fastjson:1.1.70.android'
```

注意要记得创建对象的JavaBean类；要求json对象中的key的名称与Java对象的JavaBean类中的属性名要相同，否则解析不成功！ 

- **用Fastjson解析JSON对象** 

```java
ShopInfo shopInfo = JSON.parseObject(json, ShopInfo.class);
```

- **用Fastjson解析JSON数组** 

```java
List<ShopInfo> shops = JSON.parseArray(jsons, ShopInfo.class);
```
