---
id: Tomcat
title: Tomcat
sidebar_label: Tomcat
description: Tomcat
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## JavaWeb的概念

### 什么是JavaWeb

JavaWeb 是指，所有通过Java 语言编写可以通过浏览器访问的程序的总称，叫JavaWeb。

JavaWeb 是基于请求和响应来开发的。

### 什么是请求

请求是指客户端给服务器发送数据，叫请求Request。

### 什么是响应

响应是指服务器给客户端回传数据，叫响应Response。

### 请求与响应的关系

请求和响应是成对出现的，有请求就有响应。

![未命名文件](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png)

## web资源的分类

web 资源按实现的技术和呈现的效果的不同，又分为静态资源和动态资源两种。
静态资源： html、css、js、txt、mp4 视频, jpg 图片

动态资源： jsp 页面、Servlet 程序

## 常用的web服务

Tomcat：由Apache 组织提供的一种Web 服务器，提供对jsp 和Servlet 的支持。它是一种轻量级的javaWeb 容器（服务器），也是当前应用最广的JavaWeb 服务器（免费）。

Jboss：是一个遵从JavaEE 规范的、开放源代码的、纯Java 的EJB 服务器，它支持所有的JavaEE 规范（免费）。

GlassFish： 由Oracle 公司开发的一款JavaWeb 服务器，是一款强健的商业服务器，达到产品级质量（应用很少）。

Resin：是CAUCHO 公司的产品，是一个非常流行的服务器，对servlet 和JSP 提供了良好的支持，
性能也比较优良，resin 自身采用JAVA 语言开发（收费，应用比较多）。

WebLogic：是Oracle 公司的产品，是目前应用最广泛的Web 服务器，支持JavaEE 规范，
而且不断的完善以适应新的开发要求，适合大型项目（收费，用的不多，适合大公司）。

## Tomcat 服务器和Servlet 版本的对应关系

当前企业常用的版本7.*、8.*
Servlet 程序从2.5 版本是现在世面使用最多的版本（xml 配置）
到了Servlet3.0 之后。就是注解版本的Servlet 使用。
以2.5 版本为主线讲解Servlet 程序。

## Tomcat的使用

### 安装

找到你需要用的Tomcat 版本对应的zip 压缩包，解压到需要安装的目录即可。

### 目录介绍

bin 专门用来存放Tomcat 服务器的可执行程序
conf 专门用来存放Tocmat 服务器的配置文件
lib 专门用来存放Tomcat 服务器的jar 包
logs 专门用来存放Tomcat 服务器运行时输出的日记信息
temp 专门用来存放Tomcdat 运行时产生的临时数据
webapps 专门用来存放部署的Web 工程。
work 是Tomcat 工作时的目录，用来存放Tomcat 运行时jsp 翻译为Servlet 的源码，和Session 钝化的目录。

### 启动

找到Tomcat 目录下的bin 目录下的startup.bat 文件，双击，就可以启动Tomcat 服务器。
如何测试Tomcat 服务器启动成功？？？
打开浏览器，在浏览器地址栏中输入以下地址测试：
1、http://localhost:8080
2、http://127.0.0.1:8080
3、http://真实ip:8080
当出现如下界面，说明Tomcat 服务器启动成功！！！

#### 方式2

1. 打开命令行
2. cd 到你的Tomcat 的bin 目录下
3. 敲入启动命令： catalina run

### 停止

1. 点击tomcat 服务器窗口的x 关闭按钮
2. 把Tomcat 服务器窗口置为当前窗口，然后按快捷键Ctrl+C
3. 找到Tomcat 的bin 目录下的shutdown.bat 双击，就可以停止Tomcat 服务器

### 修改Tomcat 的端口号

Mysql 默认的端口号是：3306
Tomcat 默认的端口号是：8080
找到Tomcat 目录下的conf 目录，找到server.xml 配置文件。

### 部署web工程到Tomcat中

第一种部署方法：只需要把web 工程的目录拷贝到Tomcat 的webapps 目录下即可。

1. 在webapps目录下创建一个book工程：
2. 将内容拷贝进去

第二种部署方法：找到Tomcat 下的conf 目录\Catalina\localhost\ 下,创建如下的配置文件：

abc.xml 配置文件内容如下：

```
<!-- Context 表示一个工程上下文
path 表示工程的访问路径:/abc
docBase 表示你的工程目录在哪里
-->
<Context path="/abc" docBase="E:\book" />
```

访问这个工程的路径如下:http://ip:port/abc/ 就表示访问E:\book 目录

### 手托html 页面到浏览器和在浏览器中输入http://ip:端口号/工程名/访问的区别

手托html页面的原理：
![image-20220919093042089](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220919093042089.png)

输入访问地址访问的原因

![image-20220919093127473](https://zrsaber-blog.oss-cn-hangzhou.aliyuncs.com/img/image-20220919093127473.png)

### ROOT 的工程的访问，以及默认index.html 页面的访问

当我们在浏览器地址栏中输入访问地址如下：
http://ip:port/ => 没有工程名的时候，默认访问的是ROOT 工程。
当我们在浏览器地址栏中输入的访问地址如下：
http://ip:port/工程名/ => 没有资源名，默认访问index.html 页面

## idea整合Tomcat

https://www.cnblogs.com/best/p/8543022.html

操作的菜单如下：File | Settings | Build, Execution, Deployment | Application Servers
配置

可以参考一篇博客，这个现在仅仅需要了解。
