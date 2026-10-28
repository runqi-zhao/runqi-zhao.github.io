---
id: flyweight模式
title: flyweight模式
sidebar_label: flyweight模式
description: 抽象享元模式
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Flyweight模式

<font color="red">Flyweight模式的目的就是“通过尽量的共享实例来避免new出实例”。</font>

## 例子

现在有一个将许多普哦那个字符组合成为“大型字符”的类，它的实例就是重实例。为了进行测试，我们以文件形式保存了大型字符'0'~'9'和'-'的字符具体数据。

下表展示了我们需要使用的类

| 名字           | 说明                                      |
| -------------- | ----------------------------------------- |
| BigChar        | 表示“大型字符”的类                        |
| BigCharFactory | 表示生成和共用BigChar类的实例的类         |
| BigString      | 表示生成多个BigChar组成的“大型字符串”的类 |
| Main           | 进行测试                                  |

BigChar是表示“大型字符”的类。它会从文件中读取大型字符的字体数据，并将它们保存在内存中，然后使用print方法输出大型字符。大型字符会消耗很多内存，因此我们需要考虑如何共享BigChar类的实例。

BigCharFactory类会根据需要生成BigChar类的实例。不过如果它发现之前已经生成了某个大型字符的BigChar类的实例，则会直接利用该实例，而不会再生成新的实例。生成的实例全部被保存在pool字段中。此外，为了能够快速查找出之前是否已经生成了某个大型字符所对应
的实例，我们使用了java.util.Hashmap类。

BigString类用于将多个BigChar组成“大型字符串”。

Main类是用于测试程序行为的类。

![image-20221024102539776](./assets/image-20221024102539776-1666578340776-1.png)

BigChar类表示“大型字符的类”

它的构造函数会生成接收到的字符所对应的“大型字符”版本的实例，并将其保存在fontdata字段中。

```java
public class BigChar {
    //字符名字
    private char charname;
    //大型字符对应的字符串(由'#''.''\n'组成）
    private String fontdata;

    public BigChar(char charname) {
        this.charname = charname;
        try {
            //从文件中读取大型字符的字符串
            BufferedReader reader = new BufferedReader(new FileReader("big" + charname + ".txt"));
            String line;
            StringBuffer buffer = new StringBuffer();
            while ((line = reader.readLine()) != null) {
                buffer.append(line);
                buffer.append("\n");
            }
            reader.close();
        } catch (Exception e) {
            this.fontdata = charname + "?";
        }
    }

    public void print() {
        System.out.println(fontdata);
    }

}
```

BigFactory类是生成BigChar类的实力工厂。它实现了共享实例的功能。

pool字段用于管理已经生成的BigChar类的实例。Pool有泳池的意思。现在任何存放某些东西的地方都可以被叫作Pool。泳池存储的是水，而BigCharFactory的pool中存储的则是已经生成的BigChar类的实例。

此处使用单例模式保证只生成唯一实例。

```java
public class BigCharFactory {
    private HashMap pool = new HashMap();
    private static BigCharFactory singleton = new BigCharFactory();

    private BigCharFactory() {

    }

    public static BigCharFactory getInstance() {
        return singleton;
    }

    public synchronized BigChar getBigChar(char charname) {
        BigChar bc = (BigChar) pool.get("" + charname);
        if (bc == null) {
            bc = new BigChar(charname);
            pool.put("" + charname, bc);
        }
        return bc;
    }
}
```

此处为什么要使用synchronized关键字修饰getBigChar方法？<font color="red">当多个线程几乎同时调用该方法时，在判断是否已经生成实例时可能会出错，导致new出多个实例。</font>

BigString类表示由BigChar组成的“大型字符串”的类。

```java
public class BigString {
    //大型字符的数组
    private BigChar[] bigChars;

    public BigString(String string) {
        bigChars = new BigChar[string.length()];
        BigCharFactory factory = BigCharFactory.getInstance();
        for (int i = 0; i < bigChars.length; i++) {
            bigChars[i] = factory.getBigChar(string.charAt(i));
        }
    }

    public void print() {
        for (int i = 0; i < bigChars.length; i++) {
            bigChars[i].print();
        }
    }
}
```

![image-20221024103420510](./assets/image-20221024103420510-1666578861780-3.png)

Main类进行测试

```java
public class Main {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println("Usage : java Main digits");
            System.out.println("Example : java Main 1212123");
            System.exit(0);
        }

        BigString bs = new BigString(args[0]);
        bs.print();
    }
}
```

![image-20221024103545966](./assets/image-20221024103545966-1666578946696-5.png)

## 登场角色

### Flyweight（轻量级）

按照通常方式编写程序会导致程序变重，所以如果能够共享实例会比较好，而Flyweight角色表示的就是那些实例会被共享的类。在示例程序中，由BigChar类扮演此角色。

### FlyweightFactory（轻量级工厂）

FlyweightFactory角色是生成Flyweight角色的工厂。在工厂中生成Flyweight角色可以实现共享实例。在示例程序中，由BigCharFactory类扮演此角色。

### Client（请求者）

Client角色使用FlyweightFactory角色来生成Flyweight角色。在示例程序中，由BigString类扮演此角色。

> 注意:本章中的角色划分方法与GoF书（请参见附录E[GoF])有些不同。在GoF书中，出现
> 了ConcreteFlyweight角色和UnsharedConcreteFlyweight角色，其中的ConcreteFlyweight角色
> 相当于本书中的Flyweight角色，而UnsharedConcreteFlyweight角色则没有出现在本章的示
> 例程序中。

## 总结

### 多个地方产生影响

Flyweight模式的主题是“共享”。那么，在共享实例时应当注意什么呢？

首先要想到的是“如果要改变被共享的对象，就会对多个地方产生影响”。也就是说，一个实例的改变会同时反映到所有使用该实例的地方。例如，假设我们改变了示例程序中BigChar类的'3'所对应的字体数据，那么BigString类中使用的所有·3'的字体（形状）都会发生改变。

在编程时，像这样修改一个地方会对多个地方产生影响并非总是不好。有些情况下这是好事，有些情况下这是坏事。不管怎样，“修改一个地方会对多个地方产生影响，这就是共享的特点。

因此，在决定Flyweight角色中的字段时，需要精挑细选。只将那些真正应该在多个地方共享的字段定义在Flyweight角色中即可。

### Intrinsic与Extrinsic

前面讲到的“应当共享的信息和不应当共享的信息”是有专有名词的。

**应当共享的信息被称作Intrinsic信息。**Intrinsic的意思是“本质的”“固有的”。换言之，它指的是不论实例在哪里、不论在什么情况下都不会改变的信息，或是不依赖于实例状态的信息。在示例程序中，BigChar的字体数据不论在BigString中的哪个地方都不会改变。因此，BigChar的字体数据属于Intrinsic信息。

**另一方面，不应当共享的信息被称作Extrinsic信息**。Extrinsic的意思是“外在的”“非本质的”。也就是说，它是当实例的位置、状况发生改变时会变化的信息，或是依赖于实例状态的信息。在示例程序中，BigChar的实例在BigString中是第几个字符这种信息会根据BigChar在BigStr1ng中的位置变化而发生变化。因此，不应当在BigChar中保存这个信息，它属于Extrinsic信息。

### 不要让被共享的实例被垃圾回收器回收了

java中有自动回收机制，注意不要让共享对象在长期运行的时候被回收。

要想让实例可以被垃圾回收器回收掉，只需要显式地将其置于管理对象外即可。例如，只要我们从HashMap中移除该实例的Enty,就删除了对该实例的引用。

### 内存之外的其他资源

在示例程序中，我们了解到共享实例可以减少内存使用量。一般来说，共享实例可以减少所需资源的使用量。这里的资源指的是计算机中的资源，而内存是资源中的一种。
时间也是一种资源。使用new关键字生成实例会花费时间。通过Flyweight模式共享实例可以减少使用new关键字生成实例的次数。这样，就**可****以提高程序运行速度**。

文件句柄（文件描述符）和窗口句柄等也都是一种资源。在操作系统中，可以同时使用的文件句柄和窗口句柄是有限制的。因此，如果不共享实例，应用程序在运行时很容易就会达到资源极限而导致崩溃。

## 相关设计模式

- Proxy模式
- Composite模式
- Singleton模式

