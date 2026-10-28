---
id: Cookie
title: Cookie
sidebar_label: Cookie
description: Cookie
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## 概念

1. Cookie 翻译过来是饼干的意思。
2. Cookie 是服务器通知客户端保存键值对的一种技术。
3. 客户端有了Cookie 后，每次请求都发送给服务器。
4. 每个Cookie 的大小不能超过4kb

## 如何创建Cookie

![image-20220924153030494](./assets/image-20220924153030494.png)

Servlet 程序中的代码：

```java
protected void createCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
IOException {
//1 创建Cookie 对象
Cookie cookie = new Cookie("key4", "value4");
//2 通知客户端保存Cookie
resp.addCookie(cookie);
//1 创建Cookie 对象
Cookie cookie1 = new Cookie("key5", "value5");
//2 通知客户端保存Cookie
resp.addCookie(cookie1);
resp.getWriter().write("Cookie 创建成功");
}
```

## 服务器如何获取Cookie

服务器获取客户端的Cookie 只需要一行代码：req.getCookies():Cookie[]

![image-20220924153335012](./assets/image-20220924153335012.png)

Cookie 的工具类：

```java
public class CookieUtils {
    /**
     * 查找指定名称的Cookie对象
     * @param name
     * @param cookies
     * @return
     */
    public static Cookie findCookie(String name , Cookie[] cookies){
        if (name == null || cookies == null || cookies.length == 0) {
            return null;
        }

        for (Cookie cookie : cookies) {
            if (name.equals(cookie.getName())) {
                return cookie;
            }
        }

        return null;
    }

}
```

Servlet 程序中的代码：

```java
protected void getCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
IOException {
    Cookie[] cookies = req.getCookies();
    for (Cookie cookie : cookies) {
        // getName 方法返回Cookie 的key（名）
        // getValue 方法返回Cookie 的value 值
        resp.getWriter().write("Cookie[" + cookie.getName() + "=" + cookie.getValue() + "] <br/>");
        }
        Cookie iWantCookie = CookieUtils.findCookie("key1", cookies);
            // for (Cookie cookie : cookies) {
            // if ("key2".equals(cookie.getName())) {
                // iWantCookie = cookie;
                // break;
            // }
        // }
        // 如果不等于null，说明赋过值，也就是找到了需要的Cookie
        if (iWantCookie != null) {
        resp.getWriter().write("找到了需要的Cookie");
    }
}
```

## Cookie值的修改

方案一：

1. 先创建一个要修改的同名（指的就是key）的Cookie 对象
2. 在构造器，同时赋于新的Cookie 值。
3. 调用response.addCookie( Cookie );

```java
// 方案一：
// 1、先创建一个要修改的同名的Cookie 对象
// 2、在构造器，同时赋于新的Cookie 值。
Cookie cookie = new Cookie("key1","newValue1");
// 3、调用response.addCookie( Cookie ); 通知客户端保存修改
resp.addCookie(cookie);
```

方案二：

1. 先查找到需要修改的Cookie 对象
2. 调用setValue()方法赋于新的Cookie 值。
3. 调用response.addCookie()通知客户端保存修改

```java
// 方案二：
// 1、先查找到需要修改的Cookie 对象
Cookie cookie = CookieUtils.findCookie("key2", req.getCookies());
if (cookie != null) {
// 2、调用setValue()方法赋于新的Cookie 值。
cookie.setValue("newValue2");
// 3、调用response.addCookie()通知客户端保存修改
resp.addCookie(cookie);
}
```

## Cookie生命控制

Cookie 的生命控制指的是如何管理Cookie 什么时候被销毁（删除）
setMaxAge()
正数，表示在指定的秒数后过期
负数，表示浏览器一关，Cookie 就会被删除（默认值是-1）
零，表示马上删除Cookie

```java
/**
     * 设置存活1 个小时的Cookie
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    protected void life3600(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie  = new Cookie("life3600", "life3600");
        cookie.setMaxAge(60 * 60);
        resp.addCookie(cookie);
        resp.getWriter().write("已经创建一个存活一小时得Cookie");

    }

    /**
     * 立马删除cookie
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    protected void deleteNow(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
            IOException {
        Cookie cookie = CookieUtils.findCookie("key4", req.getCookies());
        if (cookie != null) {
            cookie.setMaxAge(0);
            resp.addCookie(cookie);
            resp.getWriter().write("key4得cookie已经被删除");
        }
    }

    void defaultLife(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
            IOException {
        Cookie cookie = new Cookie("defaultLife", "defaultLife");
        cookie.setMaxAge(-1);
        resp.addCookie(cookie);
    }
```

## Cookie有效路径Path的设置

Cookie 的path 属性可以有效的过滤哪些Cookie 可以发送给服务器。哪些不发。

path 属性是通过请求的地址来进行有效的过滤。

CookieA path=/工程路径

CookieB path=/工程路径/abc

请求地址如下：

http://ip:port/工程路径/a.html

CookieA 发送

CookieB 不发送

http://ip:port/工程路径/abc/a.html

CookieA 发送

CookieB 发送

```java
protected void testPath(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
            IOException {
        Cookie cookie = new Cookie("path1","path1");
        cookie.setPath(req.getContextPath() + "/abc");
        resp.addCookie(cookie);
        resp.addCookie(cookie);
        resp.getWriter().write("创建一个带有Path路径得Cookie");
    }
```

## Cookie 练习---免输入用户名登录

![image-20220926105921640](./assets/image-20220926105921640.png)

login.jsp 页面

```jsp
<form action="http://localhost:8080/tomcat/loginServlet" method="get">
    用户名：<input type="text" name="username" value="${cookie.username.value}"> <br>
    密码：<input type="password" name="password"> <br>
    <input type="submit" value="登录">
</form>
```

LoginServlet代码：

```java
public class LoginServlet extends BaseServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
            IOException {
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        if ("runqi".equals(username) && "123456".equals(password)) {
            Cookie cookie = new Cookie("username", username);
            cookie.setMaxAge(60 * 60 * 24 * 7);
            resp.addCookie(cookie);
            System.out.println("login success");
        } else {
            System.out.println("login failed");
        }
    }
}
```

