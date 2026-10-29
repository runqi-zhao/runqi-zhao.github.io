---
id: session
title: session
sidebar_label: session
description: session
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## 概念

1. Session 就一个接口（HttpSession）。
2. Session 就是会话。它是用来维护一个客户端和服务器之间关联的一种技术。
3. 每个客户端都有自己的一个Session 会话。
4. Session 会话中，我们经常用来保存用户登录之后的信息。

## 如何创建Session和获取

如何创建和获取Session。它们的API 是一样的。

request.getSession()

第一次调用是：创建Session 会话

之后调用都是：获取前面创建好的Session 会话对象。

isNew(); 判断到底是不是刚创建出来的（新的）

true 表示刚创建

false 表示获取之前创建

每个会话都有一个身份证号。也就是ID 值。而且这个ID 是唯一的。

getId() 得到Session 的会话id 值。

## Session域数据的存取

```java
 /**
     * 往Session种保存数据
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    protected void setAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.getSession().setAttribute("key1", "value1");
        resp.getWriter().write("已经往Session种保存了数据");
    }

    /**
     * 从session种获取数据
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    protected void getAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Object attribute = req.getSession().getAttribute("key1");
        resp.getWriter().write("从Session种获取key1的数据是：" + attribute);
    }
```

## Session生命周期控制

public void setMaxInactiveInterval(int interval) 设置Session的超时时间（以秒为单位），超过指定的时长，Session
就会被销毁。
值为正数的时候，设定Session的超时时长。
负数表示永不超时（极少使用）
public int getMaxInactiveInterval()获取Session 的超时时间
public void invalidate() 让当前Session 会话马上超时无效。
Session 默认的超时时长是多少！
Session 默认的超时时间长为30 分钟。
因为在Tomcat 服务器的配置文件web.xml 中默认有以下的配置，它就表示配置了当前Tomcat服务器下所有的Session
超时配置默认时长为：30 分钟。

```xml
<session-config>
	<session-timeout>30</session-timeout>
</session-config>
```

如果说。你希望你的web 工程，默认的Session 的超时时长为其他时长。你可以在你自己的web.xml 配置文件中做以上相同的配置。就可以修改你的web 工程所有Seession 的默认超时时长。

```xml
<session-config>
	<session-timeout>20</session-timeout>
</session-config>
```

如果你想只修改个别Session 的超时时长。就可以使用上面的API。setMaxInactiveInterval(int interval)来进行单独的设
置。

session.setMaxInactiveInterval(int interval)单独设置超时时长。

Session 超时的概念介绍：

![image-20220926112007693](./assets/image-20220926112007693.png)

示例代码：

```java
protected void life3(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        HttpSession session = req.getSession();
        session.setMaxInactiveInterval(3);
        resp.getWriter().write("当前Session已经设置为3秒后超时");
    }
```

Session 马上被超时示例：

```java
protected void deleteNow(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
            IOException {
        HttpSession session = req.getSession();
        session.invalidate();
        resp.getWriter().write("Session 已经设置为超时（无效）");
    }
```

## 浏览器和Session之间关联的技术内 幕

Session 技术，底层其实是基于Cookie 技术来实现的。

![image-20220926112446034](./assets/image-20220926112446034.png)
