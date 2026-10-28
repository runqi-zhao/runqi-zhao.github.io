---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## final域的重排序规则

编译器和处理器会遵循连个重排序规则

1. 在构造函数内对一个final域写入，与随后把这个构造对象的引用复制给一个引用变量，这两个操作之间不能重排序。
2. 初次读一个包含final域的对象引用，与随后初次读这个final域，这两个操作之间不能重排序。

```java
public class FinalExample {
    int i;//普通变量
    final int j;//final变量
    static FinalExample obj;
    public FinalExample() {//构造函数
        i = 1;//写普通域
        j = 2;//写final域
    }

    public static void writer() {//写线程A执行
        obj = new FinalExample();//1
    }

    public static void reader() {//读线程B执行
        FinalExample object = obj;//都对象引用
        int a = object.i;//读普通域
        int b = object.j;//读final域
    }
}
```

假设一个线程A执行writer()方法，线程B执行reader()方法。下面两节将会用这个例子说明。

## 写final域的重排序规则

写final域的重排序规则禁止把final域的写重排序到构造函数之外。具体是2个方面：

1. JMM禁止编译器把final域的写重排序到构造函数之外。
2. 编译器会在final域写之后，构造函数return之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外。

例如上面的writer()方法。运行1时有连个步骤，如下：

1. 构造一个FinalExample类型的对象。
2. 把这个对象的引用赋值给引用变量obj。

如果线程B读对象引用与读对象成员域之间没有重排序，下图可能是一种执行时序。

![image-20221211105355001](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221211105355001.png)

<center>线程执行时序图</center>

写普通域的操作被编译器排到了构造函数之外，读线程B错误的读取了普通变量的值。而写final域的操作，被写final域的重排序规则“限定”在了构造函数之内，读线程B正确地读取了final变量初始化之后的值。

因此，写final域的重排序规则可以确保：<font color="red" size="7px">在对象引用为任意线程可见之前，对象的final域已经初始化完成。而普通域不具有这个保障。</font>

## 读final域的重排序规则

1. 在一个线程中，初次读对象引用与初次读该对象包含的final域，JMM禁止<font color="blue">处理器</font>重排序这连个操作。
2. 编译器会在读final域操作前面插入一个LoadLoad屏障。

初次读对象引用与初次读该对象包含的final域，这两个操作之间存在间接依赖关系。。由于编译器遵守间接依赖关系，因此编译器不会重排序这两个操作。大多数处理器也会遵守间接依赖，也不会重排序这两个操作。但有少数处理器允许对存在间接依赖关系的操作做重排序（比如alpha处理器），这个规则就是专门用来针对这种处理器的。
reader()方法包含3个操作。

- 初次读引用变量obj。
- 初次读引用变量obj指向对象的普通域j。
- 初次读引用变量obj指向对象的final域i。

写线程A没有发生任何重排序，同时程序在不遵守间接依赖的处理器上执行，则会有下面这种情况：

![image-20221211111006408](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221211111006408.png)

<center>线程执行时序图</center>

读对象的普通域操作被处理器重排序到读对象引用之前。读普通域之前，该域还没有被写线程A写入，会得到一个错误的结果。读final域的重排序规则会把这个读对象final域的操作“限定”在读对象引用之后，此时该final域已经被A线程初始化过了，这是一个正确的读取操作。

因此，读final域的重排序规则可以确保：

<font color="red">在读一个对象的final域之前，一定会先读包含这个final域的对象的引用。</font>

## final域为引用类型

```java
public class FinalReferenceEscapeExample {
    final int i;
    static FinalReferenceEscapeExample obj;
    public  FinalReferenceEscapeExample() {
        i = 1;      //写final域
        obj =this; //this引用在此逸出
    }

    public static void writer() {
        new FinalReferenceEscapeExample();
    }

    public static void reader() {
        if (obj != null) {
            int temp = obj.i;
        }
    }
}
```

对于引用类型，写final域的重排序规则对编译器和处理器增加了如下约束：

<font color="red">在构造函数内对一个final引用的对象的成员域写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。</font>

假设首先线程A执行writerOne()方法，执行完后线程B执行writerTwo()方法，执行完后线程C执行reader()方法。

![image-20221211113156852](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221211113156852.png)

<center>引用型final的执行时序图</center>

1是对final的写入，2是对这个final域引用的对象的成员域的写入，3是把被构造的对象的引用赋值给某个引用变量。1、3；2、3都不能重排序。

JMM可以确保读线程C至少能看到写线程A在构造函数中对final引用对象的成员域的写入。即C至少能看到数组下标0的值为1。而写线程B对数组元素的写入，读线程C可能看得到，也可能看不到。JMM不保证线程B的写入对读线程C可见，因为写线程B和读线程C之间存在数据竞争，此时的执行结果不可预知。

如果想要确保读线程C看到写线程B对数组元素的写入，写线程B和读线程C之间需要使用同步原语（lock或volatile）来确保内存可见性。

## 为什么final引用不能从构造函数内“溢出”

final域的重排序规则可以确保：在引用变量为任意线程可见之前，该引用变量指向的对象的final域已经在构造函数中被正确初始化了。为了实现这个效果，还需要保证：在构造函数内部，不能让这个被构造对象的引用为其他线程所见，也就是对象引用不能在构造函数中“逸出”。

```java
public class FinalReferenceEscapeExample {
    final int i;
    static FinalReferenceEscapeExample obj;
    public  FinalReferenceEscapeExample() {
        i = 1;      //1
        obj =this; //2
    }

    public static void writer() {
        new FinalReferenceEscapeExample();
    }

    public static void reader() {
        if (obj != null) { //3
            int temp = obj.i; //4
        }
    }
}
```

假设线程A执行writer()方法，线程B执行reader()方法。这里的操作2使得对象还未完成构造前就为线程可见。即使这里的操作2时构造函数的最后一步，且在程序中操作2排在操作1后面，执行reader()方法的线程仍然可能无法看到final域被初始化后的值，因为这里的操作1和操作2之间可能重排序。如下图所示：

![image-20221212085641928](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212085641928.png)

<center>多线程执行时序图</center>

在构造函数之返回前，被构造对象的引用不能为其他线程所见，因为此时的final域可能还没有被初始化。在构造函数返回后，任意线程都将保证能看到final域正确初始化后的值。

## final语义在处理器中的实现

以x86为例，说明final语义在处理器中的具体实现。

之前提过，写final域的重排序规则会要求编译器在final域的写之后，构造含税return之前插入一个StoreStore屏障。读final域的重排序规则要求编译器在读final域的操作前面插入一个LoadLoad屏障。

由于x86处理器不存在写-写操作做重排序，也不会存在读-读操作重排序，因此，在x86处理器中，final域的读/写不会插入任何内存屏障。

## JSR-133为什么要增强final语义

原本的final域缺陷：线程可能看到final域的改变。

增强：通过final域增加写和读重排序规则，可以为java程序员提供初始化安全保证：只要对象是正确构造的（被构造对象的引用在构造函数中没有“逸出”），那么不需要使用同步（指lock和volatile的使用）就能保证任意线程都能看到这个final域在构造函数中被初始化之后的值。
