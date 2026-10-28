---
id: proxy
title: proxy模式
sidebar_label: proxy模式
description: 代理模式 
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Proxy模式

Proxy模式在面向对象编程中，“本人”和“代理人”都是对象。如果“本人”对象太忙了，有些工作无法自己亲自完成，就将其交给“代理人”对象负责。

## 例子

本例中，示例程序实现一个“带名字的打印机”。主要目的是将文字显示在界面上。在Main类中生成PrinterProxy类的实例（即“代理人”）。首先我们会给实例赋予名字Alice并在界面中显示盖面工资。接着会将实例中的名字改为Bob，然后显示该名字。在设置和获取名字时，都不会生成真正的Printer类的实例（及本人），而是由PrinterProxy类代理。最后，知道我们调用Print方法，开始进入实际打印阶段后，Printer类才会生成Printer类的实例。

为了让PrinterProxy类与Printer类具有一致性，我们定义了Printable接口。示例程序的前提是“生成Printer类的实例”这一处理需要花费很多时间。为了在程序中体现这一点，我们在Printer类的构造函数中调用了heavyJob方法，让它干一些“重活”（虽说是重活，也不过是让程序睡眠5秒钟)。

| 名字         | 说明                             |
| ------------ | -------------------------------- |
| Printer      | 表示带名字的打印机的类（本人）   |
| Printerable  | Printer和PrinterProxy的共同接口  |
| PrinterProxy | 表示带名字的打印机的类（代理人） |
| Main         | 测试                             |

![image-20221026101143160](./assets/image-20221026101143160-1666750304878-1.png)

![image-20221026101201484](./assets/image-20221026101201484-1666750322773-3.png)

Printer类是表示“本人的类”。

```java
public class Printer implements Printable {
    private String name;
    public Printer() {
        heavyJob("正在生成Printer的实例");
    }

    public Printer(String name) {
        this.name = name;
        heavyJob("正在生成Printer的实例(" + name + ")");
    }

    @Override
    public void setPrinterName(String name) {
        this.name = name;
    }

    @Override
    public String getPrinterName() {
        return name;
    }

    @Override
    public void print(String string) {
        System.out.println("===" + name + "===");
        System.out.println(string);
    }

    private void heavyJob(String msg) {
        System.out.print(msg);
        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }

            System.out.print(".");
        }
        System.out.println("结束。");
    }
}

```

Printable接口（代码清单2l-2)用于使Printerproxy类和Printer类具有一致性。setprinterName方法用于设置打印机的名字；getprinterName用于获取打印机的名字；print用于显示文字（打印输出）。

```java
public interface Printable {
    public abstract void setPrinterName(String name);
    public abstract String getPrinterName();
    public abstract void print(String string);
}
```

PrinterProxy类是扮演“代理人”角色的类，它实现了Printable接口。

name字段中保存了打印机的名字，而real字段中保存的是“本人”。

setPrinterName方法用于设置新的打印机名字。如果real字段不为null（也就是已经生成了“本人”)，那么会设置“本人”的名字。但是当rea1字段为nu11时（即还没有生成“本人”)，那么只会设置自己（PrinterProxy的实例)的名字。

getPrinterName会返回自己的name字段。

print方法已经超出了代理人的工作范围，因此它会调用realize方法来生成本人。Realize有“实现”（使成为真的东西）的意思。在调用rea1ize方法后，real字段中会保存本人(Print类的实例)，因此可以调用real.print方法。这就是“委托”。

不论setPrinterName方法和getPrinterName方法被调用多少次，都不会生成Printer类的实例。)只有当真正需要本人时，才会生成Printer类的实例(PrinterProxy类的调用者完全不知道是否生成了本人，也不用在意是否生成了本人)。

realize方法很简单，当real字段为null时，它会使用new Printer来生成Printer类的实例；如果xeal字段不为nul1(即已经生成了本人)，则什么都不做。

**Printer类并不知道PrinterProxy类的存在**，即，Printer类并不知道自己到底通过PrinterProxy呗带哦用的还是直接呗调用的。

但反过来，PrinterProxy类是知道Printer类的。这是因为PrinterProxy类的real字段是Printer类型的。在PrinterProxy类的代码中，显式地写出了Printer这个类名。因此，PrinterProxy类是与Printer类紧密地关联在一起的组件

Printer类的setPrinterName方法和realize方法都是synchronized方法。如果没有使用修饰符synchronized,当多个线程分别调用setprinterName方法和realize方法时，可能会导致PrinterProxy类的name与Printer类的name不同。

```java
public class PrinterProxy implements Printable {
    private String name;
    private Printer real;

    public PrinterProxy() {

    }

    public PrinterProxy(String name) {
        this.name = name;
    }

    @Override
    public synchronized void setPrinterName(String name) {
        if (real != null) {
            real.setPrinterName(name);
        }
        this.name = name;
    }

    @Override
    public String getPrinterName() {
        return name;
    }

    @Override
    public void print(String string) {
        realize();
        real.print(string);
    }

    private synchronized void realize() {
        if (real == null) {
            real = new Printer(name);
        }
    }
}
```

Main类进行测试

```java
public class Main {
    public static void main(String[] args) {
        Printable p = new PrinterProxy("Alice");
        System.out.println("现在的名字是" + p.getPrinterName() + "。");
        p.setPrinterName("Bob");
        System.out.println("现在的名字是" + p.getPrinterName() + "。");
        p.print("Hello,world.");
    }
}
```

## 登场角色

### Subject（主体）

Subject角色定义了使Proxy角色和RealSubject角色之间具有一致性的接口。由于存在Subject角色，所以Client角色不必在意它所使用的究竟是Proxy角色还是RealSubject角色。在示例程序中，由Printable接口扮演此角色。

### Proxy（代理人）

Proxy角色会尽量处理来自Client角色的请求。只有当自己不能处理时，它才会将工作交给RealSubject角色。Proxy角色只有在必要时才会生成RealSubject角色。Proxy角色实现了在Subject角色中定义的接口(API)。在示例程序中，由PrinterProxy类扮演此角色。

### RealSubject（实际的主体）

“本人”RealSubject角色会在“代理人”Proxy角色无法胜任工作时出场。它与Proxy角色一样，也实现了在Subject角色中定义的接口(API)。在示例程序中，由Printer类扮演此角色。

### Client（请求者）

使用Proxy模式的角色。

![image-20221026102255113](./assets/image-20221026102255113-1666750976147-5.png)

## 总结

### 使用代理人来提升处理速度

这里可以理解成懒加载，在使用时在进行加载，从而节省时间

### 有必要划分代理人和本人吗

当然，我们也可以不划分PrinterProxy类和Printer类，而是直接在Printer类中加人惰性求值功能（即只有必要时才生成实例的功能）。不过，通过划分PrinterProxy角色和Printer角色，可以使它们成为独立的组件，在进行修改时也不会互相之间产生影响（分而治之）。

只要改变了PrinterProxy类的实现方式，即可改变在Printable接口中定义的那些方法，即对于“哪些由代理人负责处理，哪些必须本人负责处理”进行更改。而且，不论怎么改变，都不必修改Printer类。如果不想使用惰性求值功能，只需要修改Main类，将它使用new关键字生成的实例从PrinterProxy类的实例变为Printer类的实例即可。由于PrinterProxy类和Printer类都实现了Printable接口，因此Main类可以放心地切换这两个类。

在示例程序中，PrinterProxy类代表了“Proxy角色”。因此使用或是不使用PrinterProxy类就代表了使用或是不使用代理功能。

### 代理与委托

代理人只代理他能解决的问题。当遇到他不能解决的问题时，还是会“转交”给本人去解决。这里的“转交”就是在本书中多次提到过的“委托”。从PrinterProxy类的print方法中调用real.print方法正是这种“委托”的体现。

在现实世界中，应当是本人将事情委托给代理人负责，而在设计模式中则是反过来的。

### 透明性

PrinterProxy类和Printer类都实现了Printable接口，因此Main类可以完全不必在意调用的究竟是Printerproxy类还是Printer类。无论是直接使用Printer类还是通过PrinterProxy类间接地使用Printer类都可以。

在这种情况下，可以说PrinterProxy类是具有“透明性”的。就像在人和一幅画之间放置了一块透明的玻璃板后，我们依然可以透过它看到画一样，即使在Main类和Printer类之间加入一个PrinterProxy类，也不会有问题。

### HTTP代理

提到代理，许多人应该都会想到HTTP代理。HTTP代理是指位于HTTP服务器(Wb服务器)和HTTP客户端(Wb浏览器)之间，为Web页面提供高速缓存等功能的软件。我们也可以认为它是一种Proxy模式。

HTTP代理有很多功能。作为示例，我们只讨论一下它的页面高速缓存功能。通过Web浏览器访问Web页面时，并不会每次都去访问远程Web服务器来获取页面的内容，而是会先去获取HTTP代理缓存的页面。只有当需要最新页面内容或是页面的缓存期限过期时，才去访问远程Web服务器。

在这种情况下，Web服务器扮演的是Client角色，HTTP代理扮演的是Proxy角色，而web服务器扮演的则是RealSubject角色。

### 各种Proxy模式

Proxy模式由很多种变化形式

#### Vitual Proxy（虚拟代理）

Virtual Proxy就是本章中学习的Proxy模式。只有当真正需要实例时，它才生成和初始化实例。1

#### Remote Proxy（远程代理）

Remote Proxy可以让我们完全不必在意RealSubject角色是否在远程网络上，可以如同它在自己身边一样（透明性地）调用它的方法。Java的RMI（RemoteMethodInvocation:远程方法调用)就相当于Remote Proxy。

#### Access Proxy

Access Proxy用于在调用RealSubject角色的功能时设置访问限制。例如，这种代理可以只允许指定的用户调用方法，而当其他用户调用方法时则报错。

## 相关设计模式

- Adapter模式
- Decorator模式

