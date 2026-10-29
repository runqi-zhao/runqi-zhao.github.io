---
id: Servlet
title: Servlet
sidebar_label: Servlet
description: Servlet
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Servlet技术

### 概念

1. Servlet 是JavaEE 规范之一。规范就是接口
2. Servlet 就JavaWeb 三大组件之一。三大组件分别是：Servlet 程序、Filter 过滤器、Listener 监听器。
3. Servlet 是运行在服务器上的一个java 小程序，它可以接收客户端发送过来的请求，并响应数据给客户端。

### 手动实现Servlet程序

1. 编写一个类去实现Servlet 接口
2. 实现service 方法，处理请求，并响应数据
3. 到web.xml 中去配置servlet 程序的访问地址

```java
public class HelloServlet implements Servlet {

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("init");
    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    /**
     * service 方法是专门用来处理请求和响应的
     * @param servletRequest
     * @param servletResponse
     * @throws ServletException
     * @throws IOException
     */

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("3 service == Hello Servlet, it is visited");
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String method = httpServletRequest.getMethod();
        if ("GET".equals(method)) {
            doGet();
        } else if ("POST".equals(method)) {
            doPost();
        }
        System.out.println("Hello Servlet, it is visited");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {
        System.out.println("destroy");
    }

    public void doGet() {
        System.out.println("get");
        System.out.println("get");
    }

    public void doPost() {
        System.out.println("post");
        System.out.println("post");
    }
}
```

web.xml 中的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
version="4.0">
<!-- servlet 标签给Tomcat 配置Servlet 程序-->
<servlet>
<!--servlet-name 标签Servlet 程序起一个别名（一般是类名） -->
<servlet-name>HelloServlet</servlet-name>
<!--servlet-class 是Servlet 程序的全类名-->
<servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>
</servlet>
<!--servlet-mapping 标签给servlet 程序配置访问地址-->
<servlet-mapping>
<!--servlet-name 标签的作用是告诉服务器，我当前配置的地址给哪个Servlet 程序使用-->
<servlet-name>HelloServlet</servlet-name>
<!--url-pattern 标签配置访问地址<br/>
/ 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径<br/>
/hello 表示地址为：http://ip:port/工程路径/hello <br/>
-->
<url-pattern>/hello</url-pattern>
</servlet-mapping>
</web-app>
```

#### url到servlet的访问

![image-20220919185723426](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220919185723426.png)

### Servlet 的生命周期

1. 执行Servlet 构造器方法
2. 执行init 初始化方法
   第一、二步，是在第一次访问，的时候创建Servlet 程序会调用。
3. 执行service 方法
   第三步，每次访问都会调用。
4. 执行destroy 销毁方法
   第四步，在web 工程停止的时候调用。

#### GET 和POST 请求的分发处理

```java
public class HelloServlet implements Servlet {
/**
* service 方法是专门用来处理请求和响应的
* @param servletRequest
* @param servletResponse
* @throws ServletException
* @throws IOException
*/
@Override
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws
ServletException, IOException {
System.out.println("3 service === Hello Servlet 被访问了");
// 类型转换（因为它有getMethod()方法）
HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
// 获取请求的方式
String method = httpServletRequest.getMethod();
if ("GET".equals(method)) {
doGet();
} else if ("POST".equals(method)) {
doPost();
}
}
/**
* 做get 请求的操作
*/
public void doGet(){
System.out.println("get 请求");
System.out.println("get 请求");
}
/**
* 做post 请求的操作
*/
public void doPost(){
System.out.println("post 请求");
System.out.println("post 请求");
}
}
```

#### 通过继承HttpServlet 实现Servlet 程序

一般在实际项目开发中，都是使用继承HttpServlet 类的方式去实现Servlet 程序。

1. 编写一个类去继承HttpServlet 类
2. 根据业务需要重写doGet 或doPost 方法
3. 到web.xml 中的配置Servlet 程序的访问地址

Servlet 类的代码：

```java
public class HelloServlet2 extends HttpServlet {
    /**
     * doGet()在Get请求的时候调用
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("In Hello Servlet2 doGet method");
    }

    /**
     * doPost()在post请求的时候调用
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("In HelloServlet2 doPost method");
    }
}
```

web.xml 中的配置：

```xml·
<servlet>
<servlet-name>HelloServlet2</servlet-name>
<servlet-class>com.atguigu.servlet.HelloServlet2</servlet-class>
</servlet>
<servlet-mapping>
<servlet-name>HelloServlet2</servlet-name>
<url-pattern>/hello2</url-pattern>
</servlet-mapping>
```

#### Servlet 类的继承体系

![image-20220919190411286](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220919190411286.png)

## ServletConfig 类

ServletConfig 类从类名上来看，就知道是Servlet 程序的配置信息类。

Servlet 程序和ServletConfig 对象都是由Tomcat 负责创建，我们负责使用。

Servlet 程序默认是第一次访问的时候创建，ServletConfig 是每个Servlet 程序创建时，就创建一个对应的ServletConfig 对
象。

### ServletConfig 类的三大作用

1. 可以获取Servlet 程序的别名servlet-name 的值
2. 获取初始化参数init-param
3. 获取ServletContext 对象

web.xml 中的配置：

```xml
<!-- servlet 标签给Tomcat 配置Servlet 程序-->
    <servlet>
        <!--servlet-name 标签Servlet 程序起一个别名（一般是类名） -->
        <servlet-name>HelloServlet</servlet-name>
        <!--servlet-class 是Servlet 程序的全类名-->
        <servlet-class>com.zrq.tomcat.HelloServlet</servlet-class>
        <!--init-param 是初始化参数-->
        <init-param>
            <!--是参数名-->
            <param-name>username</param-name>
            <!--是参数值-->
            <param-value>root</param-value>
        </init-param>
        <!--init-param 是初始化参数-->
        <init-param>
            <!--是参数名-->
            <param-name>url</param-name>
            <!--是参数值-->
            <param-value>jdbc:mysql://localhost:3306/test</param-value>
        </init-param>
    </servlet>
    <!--servlet-mapping 标签给servlet 程序配置访问地址-->
    <servlet-mapping>
        <!--servlet-name 标签的作用是告诉服务器，我当前配置的地址给哪个Servlet 程序使用-->
        <servlet-name>HelloServlet</servlet-name>
        <!--
        url-pattern 标签配置访问地址<br/>
        / 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径<br/>
        /hello 表示地址为：http://ip:port/工程路径/hello <br/>
        -->
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
```

Servlet 中的代码：

```java
public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("init");
        System.out.println("2 init 初始化方法");
        // 1、可以获取Servlet 程序的别名servlet-name 的值
        System.out.println("HelloServlet 程序的别名是:" + servletConfig.getServletName());
        // 2、获取初始化参数init-param
        System.out.println("初始化参数username 的值是;" + servletConfig.getInitParameter("username"));
        System.out.println("初始化参数url 的值是;" + servletConfig.getInitParameter("url"));
        // 3、获取ServletContext 对象
        System.out.println(servletConfig.getServletContext());
    }
```

## ServletContext 类

### 概念

1. ServletContext 是一个接口，它表示Servlet 上下文对象
2. 一个web 工程，只有一个ServletContext 对象实例。
3. ServletContext 对象是一个域对象。
4. ServletContext 是在web 工程部署启动的时候创建。在web 工程停止的时候销毁。

什么是域对象?

域对象，是可以像Map 一样存取数据的对象，叫域对象。

这里的域指的是存取数据的操作范围，整个web 工程

|        | 存数据         | 删数据         | 删除数据           |
| ------ | -------------- | -------------- | ------------------ |
| Map    | put()          | get()          | remove()           |
| 域对象 | setAttribute() | getAttribute() | removeAttribute(); |
|        |                |                |                    |

### ServletContext 类的四个作用

1. 获取web.xml 中配置的上下文参数context-param
2. 获取当前的工程路径，格式: /工程路径
3. 获取工程部署后在服务器硬盘上的绝对路径
4. 像Map 一样存取数据

ServletContext 演示代码：

```java
@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取web.xml中配置的上下文参数context-param
        ServletContext context = getServletConfig().getServletContext();
        String username = context.getInitParameter("username");
        System.out.println("context-param 参数username的值是：" + username);
        System.out.println("context-param 参数password的值是：" + context.getInitParameter("password"));
        //2.获取当前的工程路径，格式：/工程路径
        System.out.println("当前工程路径：" + context.getContextPath());
        //3.获取工程部署后在服务器硬盘上的绝对路径
        /**
         * / 斜杠被服务器解析地址为:http://ip:port/工程名/ 映射到IDEA 代码的web 目录<br/>
         */
        System.out.println("工程部署路径是：" + context.getRealPath("/"));
        System.out.println("工程下css目录的绝对路径是：" + context.getRealPath("/css"));
        System.out.println("工程下imgs 目录1.jpg 的绝对路径是:" + context.getRealPath("/imgs/1.jpg"));
    }
```

web.xml中：

```xml
<!--context-param 是上下文参数(它属于整个web 工程)-->
<context-param>
<param-name>username</param-name>
<param-value>context</param-value>
</context-param>
<!--context-param 是上下文参数(它属于整个web 工程)-->
<context-param>
<param-name>password</param-name>
<param-value>root</param-value>
</context-param>
```

ServletContext 像Map 一样存取数据：

ContextServlet1 代码：

```java
 @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       ServletContext context = getServletContext();
        System.out.println(context);
        System.out.println("保存之前：Context获取key1的值是：" + context.getAttribute("key1"));
        context.getAttribute("key1");
        context.setAttribute("key1", "value1");
        System.out.println("Context中获取域数据key1的值是：" + context.getAttribute("key1"));
    }
```

ContextServlet2 代码：

```java
@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = getServletContext();
        System.out.println(context);
        System.out.println("Context中获取域数据key1的值是:" + context.getAttribute("key1"));
    }
```

## HTTP 协议

### 概念

什么是协议?

协议是指双方，或多方，相互约定好，大家都需要遵守的规则，叫协议。

所谓HTTP 协议，就是指，客户端和服务器之间通信时，发送的数据，需要遵守的规则，叫HTTP 协议。

HTTP 协议中的数据又叫报文。

### 请求的HTTP协议格式

客户端给服务器发送数据叫请求。

服务器给客户端回传数据叫响应。

请求又分为GET 请求，和POST 请求两种

#### GET 请求

1. 请求行
   - 请求的方式GET
   - 请求的资源路径[+?+请求参数]
   - 请求的协议的版本号HTTP/1.1
2. 请求头
   - key : value 组成不同的键值对，表示不同的含义。

![image-20220920145422646](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220920145422646.png)

#### POST请求

1. 请求行
   - 请求的方式POST
   - 请求的资源路径[+?+请求参数]
   - 请求的协议的版本号HTTP/1.1
2. 请求头
   - key : value 不同的请求头，有不同的含义
   - 空行
3. 请求体=> 就是发送给服务器的数据

![image-20220920150455631](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220920150455631.png)

#### 常用请求头的说明

Accept: 表示客户端可以接收的数据类型

Accpet-Languege: 表示客户端可以接收的语言类型

User-Agent: 表示客户端浏览器的信息

Host： 表示请求时的服务器ip 

#### 哪些是GET 请求，哪些是POST 请求和端口号

GET 请求有哪些：

1. form 标签method=get
2. a 标签
3. link 标签引入css
4. Script 标签引入js 文件
5. img 标签引入图片
6. iframe 引入html 页面
7. 在浏览器地址栏中输入地址后敲回车

POST 请求有哪些：

8. form 标签method=post

#### 响应的HTTP 协议格式

1. 响应行
   1.  响应的协议和版本号
   2. 响应状态码
   3.  响应状态描述符
2. 响应头
   1. key : v alue 不同的响应头，有其不同含义
   2. 空行
   3. 响应体-> 就是回传给客户端的数据

![image-20220920151409765](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220920151409765.png)

#### 常用的响应码说明

200 表示请求成功

302 表示请求重定向

404 表示请求服务器已经收到了，但是你要的数据不存在（请求地址错误）

500 表示服务器已经收到请求，但是服务器内部错误（代码错误）

#### MIME 类型说明

MIME 是HTTP 协议中数据类型。

MIME 的英文全称是"Multipurpose Internet Mail Extensions" 多功能Internet 邮件扩充服务。MIME 类型的格式是“大类型/小
类型”，并与某一种文件的扩展名相对应。

常见的MIME 类型：

| 文件MIME           | 类型                              |
| ------------------ | --------------------------------- |
| 超文本标记语言文本 | .html , .htm text/html            |
| 普通文本           | .txt text/plain                   |
| RTF文本            | .rtf application/rtf              |
| GIF 图形           | .gif image/gif                    |
| JPEG 图形          | .jpeg,.jpg image/jpeg             |
| au 声音文件        | .au audio/basic                   |
| MIDI 音乐文件      | mid,.midi audio/midi,audio/x-midi |
| RealAudio 音乐文件 | .ra, .ram audio/x-pn-realaudio    |
| MPEG 文件          | .mpg,.mpeg video/mpeg             |
| AVI 文件           | .avi video/x-msvideo              |
| GZIP 文件          | .gz application/x-gzip            |
| TAR 文件           | .tar application/x-tar67          |

## HttpServletRequest 类

### 作用

每次只要有请求进入Tomcat 服务器，Tomcat 服务器就会把请求过来的HTTP 协议信息解析好封装到Request 对象中。
然后传递到service 方法（doGet 和doPost）中给我们使用。我们可以通过HttpServletRequest 对象，获取到所有请求的
信息。

### 常用方法

- getRequestURI() 获取请求的资源路径
- getRequestURL() 获取请求的统一资源定位符（绝对路径）
- getRemoteHost() 获取客户端的ip 地址
- getHeader() 获取请求头
- getParameter() 获取请求的参数
- getParameterValues() 获取请求的参数（多个值的时候使用）
- getMethod() 获取请求的方式GET 或POST
- setAttribute(key, value); 设置域数据
- getAttribute(key); 获取域数据
- getRequestDispatcher() 获取请求转发对象

常用API 示例代码：

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
IOException {
    // i.getRequestURI() 获取请求的资源路径
    System.out.println("URI => " + req.getRequestURI());
    // ii.getRequestURL() 获取请求的统一资源定位符（绝对路径）
    System.out.println("URL => " + req.getRequestURL());
    // iii.getRemoteHost() 获取客户端的ip 地址
    /**
    * 在IDEA 中，使用localhost 访问时，得到的客户端ip 地址是=> 127.0.0.1<br/>
    * 在IDEA 中，使用127.0.0.1 访问时，得到的客户端ip 地址是=> 127.0.0.1<br/>
    * 在IDEA 中，使用真实ip 访问时，得到的客户端ip 地址是==> 真实的客户端ip 地址<br/>
    */
    System.out.println("客户端ip 地址=> " + req.getRemoteHost());
    // iv.getHeader() 获取请求头
    System.out.println("请求头User-Agent ==>> " + req.getHeader("User-Agent"));
    // vii.getMethod() 获取请求的方式GET 或POST
    System.out.println( "请求的方式==>> " + req.getMethod() );
}
```

### 获取请求参数

```html
<body>
<form action="http://localhost:8080/tomcat/parameterServlet" method="get">
    用户名：<input type="text" name="username"><br/>
    密码：<input type="password" name="password"><br/>
    兴趣爱好：<input type="checkbox" name="hobby" value="cpp">C++
    <input type="checkbox" name="hobby" value="java">Java
    <input type="checkbox" name="hobby" value="js">JavaScript<br/>
    <input type="submit">
</form>
</body>
```

java代码：

```java
public class ParameterServlet extends HttpServlet {
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
IOException {
// 获取请求参数
String username = req.getParameter("username");
String password = req.getParameter("password");
String[] hobby = req.getParameterValues("hobby");
System.out.println("用户名：" + username);
System.out.println("密码：" + password);
System.out.println("兴趣爱好：" + Arrays.asList(hobby));
}
}
doGet
```

#### GET请求的中文乱码解决：

```java
// 获取请求参数
String username = req.getParameter("username");
//1 先以iso8859-1 进行编码
//2 再以utf-8 进行解码
username = new String(username.getBytes("iso-8859-1"), "UTF-8");
```

#### POST请求的中文乱码解决：

```java
@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
IOException {
// 设置请求体的字符集为UTF-8，从而解决post 请求的中文乱码问题
req.setCharacterEncoding("UTF-8");
System.out.println("-------------doPost------------");
// 获取请求参数
String username = req.getParameter("username");
String password = req.getParameter("password");
String[] hobby = req.getParameterValues("hobby");
System.out.println("用户名：" + username);
System.out.println("密码：" + password);
System.out.println("兴趣爱好：" + Arrays.asList(hobby));
}
```

#### 请求的转发

什么是请求的转发?

请求转发是指，服务器收到请求后，从一次资源跳转到另一个资源的操作叫请求转发。

![image-20220921102520336](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220921102520336.png)

Servlet1代码：

```java
  @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        System.out.println("在Servlet1(柜台1)种查看参数(材料):" + username);
        req.setAttribute("key1", "柜台1的章");
        // 问路：Servlet2（柜台2）怎么走
        /**
         * 请求转发必须要以斜杠打头，/ 斜杠表示地址为：http://ip:port/工程名/ , 映射到IDEA 代码的web 目录
         * <br/>
         */
        RequestDispatcher requestDispatcher = req.getRequestDispatcher("/servlet2");
        requestDispatcher.forward(req, resp);

    }
```

Servlet2代码：

```java
@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        System.out.println("在Servlet2(柜台2)种查看参数(材料):" + username);
        Object key1 = req.getAttribute("key1");
        System.out.println(key1);
        System.out.println("柜台1是否有章：" + key1);
        System.out.println("Servlet2处理自己的业务");
    }
```

#### base 标签的作用

![image-20220921104632823](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220921104632823.png)

![image-20220921104652288](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220921104652288.png)

```html
<!DOCTYPE html>
<html lang="zh_CN">
<head>
<meta charset="UTF-8">
<title>Title</title>
<!--base 标签设置页面相对路径工作时参照的地址
href 属性就是参数的地址值
-->
<base href="http://localhost:8080/tomcat/a/b/">
</head>
<body>
这是a 下的b 下的c.html 页面<br/>
<a href="../../index.html">跳回首页</a><br/>
</body>
</html>
```

#### Web 中的相对路径和绝对路径

在javaWeb 中，路径分为相对路径和绝对路径两种：

相对路径是：

. 表示当前目录

.. 表示上一级目录

资源名表示当前目录/资源名

绝对路径：

http://ip:port/工程路径/资源路径

在实际开发中，路径都使用绝对路径，而不简单的使用相对路径。

1. 绝对路径
2. base+相对

#### web 中/斜杠的不同意义

在web 中/ 斜杠是一种绝对路径。

/ 斜杠如果被浏览器解析，得到的地址是：http://ip:port/

```html
<a href="/">斜杠</a>
```

/ 斜杠如果被服务器解析，得到的地址是：http://ip:port/工程路径

1. `<url-pattern>/servlet1</url-pattern>`
2. servletContext.getRealPath(“/”);
3. request.getRequestDispatcher(“/”);

特殊情况： response.sendRediect(“/”); 把斜杠发送给浏览器解析。得到http://ip:port/

## HttpServletResponse类

### 作用

HttpServletResponse 类和HttpServletRequest 类一样。每次请求进来，Tomcat 服务器都会创建一个Response 对象传递给Servlet 程序去使用。HttpServletRequest 表示请求过来的信息，HttpServletResponse 表示所有响应的信息，我们如果需要设置返回给客户端的信息，都可以通过HttpServletResponse 对象来进行设置。

### 两个输出流的说明

字节流	getOutputStream(); 		常用于下载（传递二进制数据）

字符流	getWriter(); 			  常用于回传字符串（常用）

两个流同时只能使用一个。

使用了字节流，就不能再使用字符流，反之亦然，否则就会报错。

### 如何往客户端回传数据

要求： 往客户端回传字符串数据。

```java
@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter writer = resp.getWriter();
        writer.write("response's content!!!");
    }
```

#### 解决中文乱码

##### 方式一

```java
// 设置服务器字符集为UTF-8
resp.setCharacterEncoding("UTF-8");
// 通过响应头，设置浏览器也使用UTF-8 字符集
resp.setHeader("Content-Type", "text/html; charset=UTF-8");
```

##### 方式二（推荐）

```java
// 它会同时设置服务器和客户端都使用UTF-8 字符集，还设置了响应头
// 此方法一定要在获取流对象之前调用才有效
resp.setContentType("text/html; charset=UTF-8");
```

### 请求重定向

请求重定向，是指客户端给服务器发请求，然后服务器告诉客户端说。我给你一些地址。你去新地址访问。叫请求重定向（因为之前的地址可能已经被废弃）。
![image-20220921110020863](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220921110020863.png)

请求重定向的第一种方案：

```java
// 设置响应状态码302 ，表示重定向，（已搬迁）
resp.setStatus(302);
// 设置响应头，说明新的地址在哪里
resp.setHeader("Location", "http://localhost:8080");
```

请求重定向的第二种方案（推荐使用）：

```java
resp.sendRedirect("http://localhost:8080");
```

