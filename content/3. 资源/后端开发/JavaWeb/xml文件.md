---
id: xml文件
title: xml文件
sidebar_label: xml文件
description: xml文件
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## 什么是xml

xml是可拓展的标记性语言。

## xml的作用

1. 用来保存数据

   ```xml
   <students>
       <student>
           <id>1</id>
           <name>华仔</name>
       </student>
       <student>
           <id>2</id>
           <name>张三</name>
       </student>
   </students>
   ```

2. 它还可以作为项目或者模块的配置文件

3. 还可以做为网络传输数据的格式（现在JSON 为主）。

## xml语法

### 文件声明

1. 建立一个xml文件

2. xml的文件开头格式必须为：

   ```xml
   <!-- xml 声明version 是版本的意思encoding 是编码-->
   <!-- 而且这个<?xml 要连在一起写，否则会有报错 --> 
   <?xml version="1.0" encoding="UTF-8"?>
   ```

   属性：

   | version             | 版本号                       |
   | ------------------- | ---------------------------- |
   | encoding            | 文件编码                     |
   | standalone="yes/no" | 表示此xml是否是独立的xml文件 |

3. 编写具体的xml文件

   ```java
   <?xml version="1.0" encoding="UTF-8" ?>
   <!-- xml 声明version 是版本的意思encoding 是编码-->
   <books>
       <book id="SN123123413241"> <!-- book 标签描述一本图书id 属性描述的是图书的编号-->
           <name>java 编程思想</name> <!-- name 标签描述的是图书的信息-->
           <author>华仔</author> <!-- author 单词是作者的意思，描述图书作者-->
           <price>9.9</price> <!-- price 单词是价格，描述的是图书的价格-->
       </book>
       <book id="SN12341235123"> <!-- book 标签描述一本图书id 属性描述的是图书的编号-->
           <name>葵花宝典</name> <!-- name 标签描述的是图书的信息-->
           <author>班长</author> <!-- author 单词是作者的意思，描述图书作者-->
           <price>5.5</price> <!-- price 单词是价格，描述的是图书的价格-->
       </book>
   </books>
   ```

### xml注释

html 和XML 注释一样: <!-- html 注释-->

### 元素（标签）

咱们先回忆一下:
html 标签：

- 格式：<标签名>封装的数据</标签名>
- 单标签: `<标签名/> <br/> 换行<hr />水平线`
- 双标签: <标签名>封装的数据</标签名>
- 标签名大小写不敏感
- 标签有属性，有基本属性和事件属性
- 标签要闭合（不闭合，html 中不报错。但我们要养成良好的书写习惯。闭合）

### xml元素

什么是xml元素？

xml元素指的是从(且包括)开始标签直到(且包括)结束标签的部分。

元素可包含其他元素、文本或者两者的混合物。元素可以拥有属性。

```java
<booksotre>
    <book category = "CHILDREN">
        <title>Harry Potter</title>
        <aythor>J K.Rowing</aythor>
        <year>2005</year>
        <price>29.99</price>
    </book>
    <book category="WRB">
        <title>Learning XML</title>
        <aythor>Erik T. Ray</aythor>
        <year>2003</year>
        <price>39.95</price>
    </book>
</booksotre>
```

上例中，`<bookstore>与<book>`都是拥有元素内容，因为它们包含了其他元素。`<author>`只有文化内容，因为它仅包含文本。

元素：指从开始标签到结束标签的内容

例如：`<title>java编程思想</title>`

元素 我们可以简单的理解为是标签。

### xml的命名规则

1. 名称可以含有字母、数字以及其他字符

   ```xml
   <book id="SN123123413241"> <!-- book 标签描述一本图书id 属性描述的是图书的编号-->
           <name>java 编程思想</name> <!-- name 标签描述的是图书的信息-->
           <author>华仔</author> <!-- author 单词是作者的意思，描述图书作者-->
           <price>9.9</price> <!-- price 单词是价格，描述的是图书的价格-->
       </book>
   ```

2. 名称不能以数字或者标点符号开始

   ![image-20220918201010859](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220918201010859.png)

3. 名称不能包含空格

### xml中的元素分类

分为单标签与双标签

单标签
格式： <标签名属性=”值” 属性=”值” ...... />
双标签
格式：< 标签名属性=”值” 属性=”值” ......>文本数据或子标签</标签名>

## xml属性

xml 的标签属性和html 的标签属性是非常类似的，属性可以提供元素的额外信息

在标签上可以书写属性：

- 一个标签上可以书写多个属性。**每个属性的值必须使用引号引起来**。
- 规则和标签的书写规则一致。

xml元素可以在开始标签中包含属性，类似于html。

属性（Attribute）提供关于元素的额外信息

### xml属性

在html，有这个：`<img src= "computer.gif"/`。“src”属性提供有关`<img>`元素的额外信息。

在html中（以及xml中），属性提供有关元素的额外信息：

```html
<img src="compuer.gif"/>
<a href="demo.asp"/>
```

属性通常提供不属于数据组成部分的信息。在下面的例子中，文件类型与数据无关，但是对需要处理这个元素软件你来说却很重要：

```xml
<file type="gif">computer.gif</file>
```

1. xml属性必须加引号

属性值必须被引号包围，不过单引号与双引号均可使用。比如一个人的性别，person标签可以这样写：

```xml
<person sex="female">
```

或者可以这样写：

```xml
<person sex='female'>
```

## 语法规则

1. 所有XML 元素都须有关闭标签（也就是闭合）

2. XML 标签对大小写敏感

3. XML 必须正确地嵌套

4. XML 文档必须有根元素

   根元素就是顶级元素，

   没有父标签的元素，叫顶级元素。

   根元素是没有父标签的顶级元素，而且是唯一一个才行。

5. XML 的属性值必须加引号

6. XML 中的特殊字符

7. 文本区域（CDATA 区）
   CDATA 语法可以告诉xml 解析器，我CDATA 里的文本内容，只是纯文本，不需要xml 语法解析
   CDATA 格式：
   `<![CDATA[ 这里可以把你输入的字符原样显示，不会解析xml ]]>`

## xml解析技术介绍

xml可拓展的标记语言。

   不管是html文件还是xml文件都是标记行文档，都可以使用w3c 组织制定的dom 技术来解析。

![image-20220918204905477](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220918204905477.png)

document 对象表示的是整个文档（可以是html 文档，也可以是xml 文档）
早期JDK 为我们提供了两种xml 解析技术DOM 和Sax 简介（已经过时，但我们需要知道这两种技术）
dom 解析技术是W3C 组织制定的，而所有的编程语言都对这个解析技术使用了自己语言的特点进行实现。

Java 对dom 技术解析标记也做了实现。

sun 公司在JDK5 版本对dom 解析技术进行升级：SAX（ Simple API for XML ）
SAX 解析，它跟W3C 制定的解析不太一样。它是以类似事件机制通过回调告诉用户当前正在解析的内容。
它是一行一行的读取xml 文件进行解析的。不会创建大量的dom 对象。
所以它在解析xml 的时候，在内存的使用上。和性能上。都优于Dom 解析。

第三方的解析：

jdom 在dom 基础上进行了封装、

**dom4j 又对jdom 进行了封装**。

pull 主要用在Android 手机开发，是在跟sax 非常类似都是事件机制解析xml 文件。

这个Dom4j 它是第三方的解析技术。我们需要使用第三方给我们提供好的类库才可以解析xml 文件。

## dom4j 解析技术（重点）

由于dom4j 它不是sun 公司的技术，而属于第三方公司的技术，我们需要使用dom4j 就需要到dom4j 官网下载dom4j
的jar 包。

### dom4j变成步骤：

第一步： 先加载xml 文件创建Document 对象
第二步：通过Document 对象拿到根元素对象
第三步：通过根元素.elelemts(标签名); 可以返回一个集合，这个集合里放着。所有你指定的标签名的元素对象
第四步：找到你想要修改、删除的子元素，进行相应在的操作
第五步，保存到硬盘上

### 获取document对象

创建一个lib 目录，并添加dom4j 的jar 包。并添加到类路径。
需要解析的books.xml 文件内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<books>
    <book sn="SN12341232">
        <name>辟邪剑谱</name>
        <price>9.9</price>
        <author>班主任</author>
    </book>
    <book sn="SN12341231">
        <name>葵花宝典</name>
        <price>99.99</price>
        <author>班长</author>
    </book>
</books>
```

解析获取Document 对象的代码
第一步，先创建SaxReader 对象。这个对象，用于读取xml 文件，并创建Document

```java
 @Test
    public void test1() throws Exception {
        // 创建一个SaxReader输入流，去读取 xml配置文件，生成Document对象
        SAXReader saxReader = new SAXReader();

        Document document = saxReader.read("src/books.xml");

        System.out.println(document);
    }
```

### 遍历标签获取所有标签中的内容

需要分四步操作:
第一步，通过创建SAXReader 对象。来读取xml 文件，获取Document 对象
第二步，通过Document 对象。拿到XML 的根元素对象
第三步，通过根元素对象。获取所有的book 标签对象
第四小，遍历每个book 标签对象。然后获取到book 标签对象内的每一个元素，再通过getText() 方法拿到起始标签和结
束标签之间的文本内容

```java
/**
     * 读取xml 文件中的内容
     * @throws DocumentException
     */
    @Test
    public void readXML() throws DocumentException {
        // 需要分四步操作:
        // 第一步，通过创建SAXReader 对象。来读取xml 文件，获取Document 对象
        // 第二步，通过Document 对象。拿到XML 的根元素对象
        // 第三步，通过根元素对象。获取所有的book 标签对象
        // 第四步，遍历每个book 标签对象。然后获取到book 标签对象内的每一个元素，再通过getText() 方法拿到起始标签和结束标签之间的文本内容
        SAXReader saxReader = new SAXReader();
        Document document = saxReader.read("src/books.xml");
        Element root = document.getRootElement();
        List<Element> books = root.elements("book");
        for (Element book : books) {
            // 测试
            // System.out.println(book.asXML());
            // 拿到book 下面的name 元素对象
            Element nameElement = book.element("name");
            // 拿到book 下面的price 元素对象
            Element priceElement = book.element("price");
            // 拿到book 下面的author 元素对象
            Element authorElement = book.element("author");
            // 再通过getText() 方法拿到起始标签和结束标签之间的文本内容
            System.out.println("书名" + nameElement.getText() + " , 价格:"
                    + priceElement.getText() + ", 作者：" + authorElement.getText());
        }
    }
```

