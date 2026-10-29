---
id: jsp
title: jsp
sidebar_label: jsp
description: jsp
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## 概念

jsp 的全换是java server pages。Java 的服务器页面。

jsp 的主要作用是代替Servlet 程序回传html 页面的数据。

因为Servlet 程序回传html 页面数据是一件非常繁锁的事情。开发成本和维护成本都极高。

Servlet 回传html 页面数据的代码：

```java
 @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        PrintWriter writer = resp.getWriter();
        writer.write("<!DOCTYPE html>\r\n");
        writer.write("<html lang=\"en\">\r\n");
        writer.write(" <head>\r\n");
        writer.write(" <meta charset=\"UTF-8\">\r\n");
        writer.write(" <title>Title</title>\r\n");
        writer.write(" </head>\r\n");
        writer.write(" <body>\r\n");
        writer.write(" 这是html 页面数据\r\n");
        writer.write(" </body>\r\n");
        writer.write("</html>\r\n");
        writer.write("\r\n");
    }
```

jsp 回传一个简单html 页面的代码：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
	<title>Title</title>
</head>
<body>
	这是html 页面数据
</body>
</html>
```

jsp 的小结：

1. 如何创建jsp 的页面?

   使用idea...即可创建

2. jsp 如何访问：

   jsp 页面和html 页面一样，都是存放在web 目录下。访问也跟访问html 页面一样。

   比如：

   在web 目录下有如下的文件：

   web 目录
   a.html 页面访问地址是=> http://ip:port/工程路径/a.html
   b.jsp 页面访问地址是=> http://ip:port/工程路径/b.jsp

## 本质

jsp 页面本质上是一个Servlet 程序。

当我们第一次访问jsp 页面的时候。Tomcat 服务器会帮我们把jsp 页面翻译成为一个java 源文件。并且对它进行编译成为.class 字节码程序。我们打开java 源文件不难发现其里面的内容是：

我们跟踪原代码发现，HttpJspBase 类。它直接地继承了HttpServlet 类。也就是说。jsp 翻译出来的java 类，它间接了继承了HttpServlet 类。也就是说，翻译出来的是一个Servlet 程序

总结：通过翻译的java 源代码我们就可以得到结果：jsp 就是Servlet 程序。
大家也可以去观察翻译出来的Servlet 程序的源代码，不难发现。其底层实现，也是通过输出流。把html 页面数据回传给客户端。

```java
public void _jspService(final javax.servlet.http.HttpServletRequest request, final
javax.servlet.http.HttpServletResponse response)
throws java.io.IOException, javax.servlet.ServletException {
final java.lang.String _jspx_method = request.getMethod();
if (!"GET".equals(_jspx_method) && !"POST".equals(_jspx_method) && !"HEAD".equals(_jspx_method)
&& !javax.servlet.DispatcherType.ERROR.equals(request.getDispatcherType())) {
response.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, "JSPs only permit GET POST or
HEAD");
return;
}
final javax.servlet.jsp.PageContext pageContext;
javax.servlet.http.HttpSession session = null;
final javax.servlet.ServletContext application;
final javax.servlet.ServletConfig config;
javax.servlet.jsp.JspWriter out = null;
final java.lang.Object page = this;
javax.servlet.jsp.JspWriter _jspx_out = null;
javax.servlet.jsp.PageContext _jspx_page_context = null;
try {
response.setContentType("text/html;charset=UTF-8");
pageContext = _jspxFactory.getPageContext(this, request, response,
null, true, 8192, true);
_jspx_page_context = pageContext;
application = pageContext.getServletContext();
config = pageContext.getServletConfig();
session = pageContext.getSession();
out = pageContext.getOut();
_jspx_out = out;
out.write("\r\n");
out.write("\r\n");
out.write("<html>\r\n");
out.write("<head>\r\n");
out.write(" <title>Title</title>\r\n");
out.write("</head>\r\n");
out.write("<body>\r\n");
out.write(" a.jsp 页面\r\n");
out.write("</body>\r\n");
out.write("</html>\r\n");
} catch (java.lang.Throwable t) {
if (!(t instanceof javax.servlet.jsp.SkipPageException)){
out = _jspx_out;
if (out != null && out.getBufferSize() != 0)
try {
if (response.isCommitted()) {
out.flush();
} else {
out.clearBuffer();
}
} catch (java.io.IOException e) {}
if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
else throw new ServletException(t);
}
} finally {
_jspxFactory.releasePageContext(_jspx_page_context);
}
}
```

## 三种语法

### jsp头部的page指令

jsp 的page 指令可以修改jsp 页面中一些重要的属性，或者行为。

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

1. language 属性表示jsp 翻译后是什么语言文件。暂时只支持java。
2. contentType 属性表示jsp 返回的数据类型是什么。也是源码中response.setContentType()参数值
3. pageEncoding 属性表示当前jsp 页面文件本身的字符集。
4. import 属性跟java 源代码中一样。用于导包，导类。
   == 两个属性是给out 输出流使用==
5. autoFlush 属性设置当out 输出流缓冲区满了之后，是否自动刷新冲级区。默认值是true。
6. buffer 属性设置out 缓冲区的大小。默认是8kb

​	缓冲区溢出错误：

![image-20220922093105425](./assets/image-20220922093105425-1663810534250-2.png)

== 两个属性是给out 输出流使用 ==

7. errorPage 属性设置当jsp 页面运行时出错，自动跳转去的错误页面路径。

   ```jsp
   <!--
   errorPage 表示错误后自动跳转去的路径<br/>
   这个路径一般都是以斜杠打头，它表示请求地址为http://ip:port/工程路径/
   映射到代码的Web 目录
   -->
   ```

8. isErrorPage 属性设置当前jsp 页面是否是错误信息页面。默认是false。如果是true 可以
   获取异常信息。
9. session 属性设置访问当前jsp 页面，是否会创建HttpSession 对象。默认是true。
10. extends 属性设置jsp 翻译出来的java 类默认继承谁。

### jsp 中的常用脚本

#### 声明脚本(极少使用)

声明脚本的格式是： <%! 声明java 代码%>

作用：可以给jsp 翻译出来的java 类定义属性和方法甚至是静态代码块。内部类等。

练习：

1. 声明类属性
2. 声明static 静态代码块
3. 声明类方法
4. 声明内部类

代码示例：

```jsp
<%--1、声明类属性--%>
<%!
private Integer id;
private String name;
private static Map<String,Object> map;
%>
<%--2、声明static 静态代码块--%>
<%!
static {
map = new HashMap<String,Object>();
map.put("key1", "value1");
map.put("key2", "value2");
map.put("key3", "value3");
}
%>
<%--3、声明类方法--%>
<%!
public int abc(){
return 12;
}
%>
<%--4、声明内部类--%>
<%!
public static class A {
private Integer id = 12;
private String abc = "abc";
}
%>
```

声明脚本代码翻译对照：
![image-20220922093616478](./assets/image-20220922093616478-1663810587216-4.png)

#### 表达式脚本（常用）

表达式脚本的格式是：<%=表达式%>

表达式脚本的作用是：的jsp 页面上输出数据。

表达式脚本的特点：

1. 所有的表达式脚本都会被翻译到`_jspService()` 方法中
2. 表达式脚本都会被翻译成为out.print()输出到页面上
3. _由于表达式脚本翻译的内容都在_jspService() 方法中,所以_jspService()方法中的对象都可以直接使用。
4. 表达式脚本中的表达式不能以分号结束。

练习：

1. 输出整型
2. 输出浮点型
3. 输出字符串
4. 输出对象

示例代码:

```jsp
<%=12 %> <br>
<%=12.12 %> <br>
<%="我是字符串" %> <br>
<%=map%> <br>
<%=request.getParameter("username")%>
```

翻译对照:

![image-20220922093940904](./assets/image-20220922093940904.png)

### 代码脚本

代码脚本的格式是：

```jsp
<%
java 语句
%>
```

代码脚本的作用是：可以在jsp 页面中，编写我们自己需要的功能（写的是java 语句）。

代码脚本的特点是：

1. 代码脚本翻译之后都在`_jspService` 方法中
2. 代码脚本由于翻译到`_jspService()`方法中，所以在_jspService()方法中的现有对象都可以直接使用。
3. 可以由多个代码脚本块组合完成一个完整的java 语句。
4. 代码脚本还可以和表达式脚本一起组合使用，在jsp 页面上输出数据

练习：

1. 代码脚本----if 语句
2. 代码脚本----for 循环语句
3. 翻译后java 文件中_jspService 方法内的代码都可以写

示例代码:

```jsp
<%--练习：--%>
<%--1.代码脚本----if 语句--%>
<%
int i = 13 ;
if (i == 12) {
%>
<h1>国哥好帅</h1>
<%
} else {
%>
<h1>国哥又骗人了！</h1>
<%
}
%>
<br>
<%--2.代码脚本----for 循环语句--%>
<table border="1" cellspacing="0">
<%
for (int j = 0; j < 10; j++) {
%>
<tr>
<td>第<%=j + 1%>行</td>
</tr>
<%
}
%>
</table>
<%--3.翻译后java 文件中_jspService 方法内的代码都可以写--%>
<%
String username = request.getParameter("username");
System.out.println("用户名的请求参数值是：" + username);
%>
```

翻译之后的对比：

![image-20220922094303062](./assets/image-20220922094303062.png)

## 三种注释

### html注释

```html
<!-- 这是html 注释-->
```

html 注释会被翻译到java 源代码中。在_jspService 方法里，以out.writer 输出到客户端。

### java注释

```jsp
<%
// 单行java 注释
/* 多行java 注释*/
%>
```

java 注释会被翻译到java 源代码中。

### jsp注释

```jsp
<%-- 这是jsp 注释--%>
```

jsp 注释可以注掉，jsp 页面中所有代码。

## 九大内置对象

jsp 中的内置对象，是指Tomcat 在翻译jsp 页面成为Servlet 源代码后，内部提供的九大对象，叫内置对象。

- pageContext 一个页面，当前页面
- request 一次请求所有被转发过的servlet
- session 一次会话所有的servlet
- application 一个项目所有的servlet
- response 相应信息，比较底层，没有做封装
- out 内置了一个缓冲区，响应信息推荐使用out
- config 配置信息，很少使用
- page 当前页面对象，基本不用
- exception 异常对象，根本不用

## 四大域对象

四个域对象分别是：

- pageContext 	(PageContextImpl 类) 	当前jsp 页面范围内有效
- request         (HttpServletRequest 类)  一次请求内有效
- session         (HttpSession 类)         一个会话范围内有效（打开浏览器访问服务器，直到关闭浏览器）
- application     (ServletContext 类)      整个web 工程范围内都有效（只要web 工程不停止，数据都在）

域对象是可以像Map 一样存取数据的对象。四个域对象功能一样。不同的是它们对数据的存取范围。

虽然四个域对象都可以存取数据。在使用上它们是有优先顺序的。

四个域在使用的时候，优先顺序分别是，他们从小到大的范围的顺序。

pageContext => request => session => application

scope.jsp 页面

```jsp
<body>
<h1>scope.jsp 页面</h1>
<%
// 往四个域中都分别保存了数据
pageContext.setAttribute("key", "pageContext");
request.setAttribute("key", "request");
session.setAttribute("key", "session");
application.setAttribute("key", "application");
%>
pageContext 域是否有值：<%=pageContext.getAttribute("key")%> <br>
request 域是否有值：<%=request.getAttribute("key")%> <br>
session 域是否有值：<%=session.getAttribute("key")%> <br>
application 域是否有值：<%=application.getAttribute("key")%> <br>
<%
request.getRequestDispatcher("/scope2.jsp").forward(request,response);
%>
</body>
```

scope2.jsp 页面

```jsp
<body>
<h1>scope2.jsp 页面</h1>
pageContext 域是否有值：<%=pageContext.getAttribute("key")%> <br>
request 域是否有值：<%=request.getAttribute("key")%> <br>
session 域是否有值：<%=session.getAttribute("key")%> <br>
application 域是否有值：<%=application.getAttribute("key")%> <br>
</body>
```

## jsp中的out输出和response.getWriter输出的区别

response 中表示响应，我们经常用于设置返回给客户端的内容（输出）out 也是给用户做输出使用的。

![image-20220922101535888](./assets/image-20220922101535888.png)

由于jsp 翻译之后，底层源代码都是使用out 来进行输出，所以一般情况下。我们在jsp 页面中统一使用out 来进行输出。避免打乱页面输出内容的顺序。

out.write() 输出字符串没有问题

out.print() 输出任意数据都没有问题（都转换成为字符串后调用的write 输出）

**深入源码，浅出结论：在jsp 页面中，可以统一使用out.print()来进行输出**

## jsp的常用标签

### jsp静态包含

示例说明：

```jsp
<%--
<%@ include file=""%> 就是静态包含
file 属性指定你要包含的jsp 页面的路径
地址中第一个斜杠/ 表示为http://ip:port/工程路径/ 映射到代码的web 目录
静态包含的特点：
1、静态包含不会翻译被包含的jsp 页面。
2、静态包含其实是把被包含的jsp 页面的代码拷贝到包含的位置执行输出。
--%>
<%@ include file="/include/footer.jsp"%>
```

### jsp动态包含

实例说明：

```jsp
<%--
<jsp:include page=""></jsp:include> 这是动态包含
page 属性是指定你要包含的jsp 页面的路径
动态包含也可以像静态包含一样。把被包含的内容执行输出到包含位置
动态包含的特点：
1、动态包含会把包含的jsp 页面也翻译成为java 代码
2、动态包含底层代码使用如下代码去调用被包含的jsp 页面执行输出。
JspRuntimeLibrary.include(request, response, "/include/footer.jsp", out, false);
3、动态包含，还可以传递参数
--%>
<jsp:include page="/include/footer.jsp">
<jsp:param name="username" value="bbj"/>
<jsp:param name="password" value="root"/>
</jsp:include>
```

动态包含的底层原理：

![image-20220922102141254](./assets/image-20220922102141254.png)

### jsp 标签-转发

示例说明：

```jsp
<%--
<jsp:forward page=""></jsp:forward> 是请求转发标签，它的功能就是请求转发
page 属性设置请求转发的路径
--%>
<jsp:forward page="/scope2.jsp">
</jsp:forward>
```

## 练习题

### 练习一

输出九九乘法表

```jsp
<head>
    <title>Title</title>
    <style type="text/css">

    table{
      width: 650px;
    }

  </style>
</head>
<body>
<h1 align="center">九九乘法口诀</h1>
<table align="center">
  <% for (int i = 1; i <= 9; i++) { %>
  <tr>
    <% for (int j = 1; j <= i; j++) { %>
    <td><%= j + "x" + i + "=" + (i * j)%></td>
    <% } %>
  </tr>
  <% } %>
</table>
</body>
```

### 练习二

jsp 输出一个表格，里面有10 个学生信息。

student类

```java
public class Student {
	private Integer id;
	private String name;
	private Integer age;
	private String phone;
```

SearchStudentServlet 程序：

```java
 @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        List<Student> studentList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            int t= i + 1;
            studentList.add(new Student(t, "name" + t, 18 + t, "phone" + t));
        }
        req.setAttribute("stuList", studentList);
        req.getRequestDispatcher("/test/showStudent.jsp").forward(req, resp);
    }
```

showStudent.jsp 页面

```jsp
<html>
<head>
    <title>Title</title>
</head>
<style>
    table {
        border: 1px blue solid;
        width: 600px;
        border-collapse: collapse;
    }

    td, th {
        border: 1px blue solid;
    }
</style>
<body>
<%
    List<Student> studentList = (List<Student>) request.getAttribute("stuList");
%>
<table>
    <tr>
        <td>编号</td>
        <td>姓名</td>
        <td>年龄</td>
        <td>电话</td>
        <td>操作</td>
    </tr>
    <tr>
        <% for (Student student : studentList) { %>
        <td><%=student.getId()%></td>
        <td><%=student.getName()%></td>
        <td><%=student.getPhone()%></td>
        <td>删除、修改</td>
    </tr>
    <% } %>
</table>
</body>
</html>
```

