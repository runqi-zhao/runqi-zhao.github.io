---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## Singleton模式

在程序运行时，通常都会生成很多实例。表示字符串的java.lang.String类的实例与字符串是一对一的关系，所以当有1000个字符串的时候，就会有1000个实例。

当我们想在程序中表示某个东西只会存在一个时，就会有“只能创建一个实例”的需求。这个时候的我们的目的就成了：

- 想确保任何情况下都绝对只有一个实例
- 想在程序上表现出“只存在一个实例”

像这样的确保只生成一个实例的模式被称作Singleton模式。

## 例子

| 名字      | 说明               |
| --------- | ------------------ |
| Singleton | 只存在一个实例的类 |
| Main      | 测试程序行为的类   |

在类图中，构造函数Singleton前面带有“-”，表示Singleton函数是private。此外，getInstance方法带有下划线，表示该方法是static方法。

![image-20221009092908183](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221009092908183-1665278949340-1.png)

Singleton类只会生成一个实例。Singleton类定义了static字段（类的成员变量）singleton，并将其初始化为Singleton类的实例。初始化行为仅在类被加载时进行一次。

Singleton类的构造函数时Private的，这是为了禁止从Singleton类外部调用构造函数。Singleton模式的作用在于**可以确保任何情况下只能生成一个实例**。为了达到这个目的，构造函数必须设置为private。

还准备了getInstance方法，以便于**程序从Singleton类外部获取Singleton类唯一的实例**。

```java
public class Singleton {
    private static Singleton singleton = new Singleton();

    private Singleton() {
        System.out.println("生成了一个实例");
    }

    public static Singleton getInstance() {
        return singleton;
    }
}
```

接下来使用Main方法进行测试，生成两个Singleton对象，判断`使用==是否返回true`（即只想同一片内存地址）

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Start.");
        Singleton singleton = Singleton.getInstance();
        Singleton singleton1 = Singleton.getInstance();
        System.out.println(singleton == singleton1);
        System.out.println("End.");
    }
}
```

## 登场角色

在Singleton有以下登场角色

- Singleton

  在Singleton模式中，只有Singleton这一个角色。Singleton角色有一个返回唯一实例的static方法。该方法总是会返回同一个实例。

![image-20221009094040463](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221009094040463-1665279641311-3.png)

## 总结

### 为什么必须要设置限制

Singleton模式对实例的数量设置了限制。设置限制恰是就是为程序增加一项前提条件。当存在多个实例时，实例之间相互影响，可能会产生意想不到的bug。但是，如果确保只有一个实例，就可以在这个前提条件下放心编程。

### 何时生成这个唯一实例

程序运行后，在第一次调用getInstance方法时，Singleton类会被初始化。也就是这个时候，static字段singleton被初始化，生成唯一的一个实例。

## 拓展：多线程问题（重中之重）

在我们的例子中，我们创建单例模式是使用饿汉模式进行创建的，会存在一个问题：无论我们是否使用，因为其为静态变量，都会在程序最开始运行时进行初始化。因此，有人提出了使用饿汉式来进行编程(**这是错误的**)：

```java
public class Singleton {
    private static Singleton singleton = null;
    
    private Singleton() {
        System.out.println("生成了一个实例");
    }
    
    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

这里会有多线程问题：多个线程同时调用Singleton，会生成多个实例！！违背了单例模式

为了确保生成多个实例，有人提出以下方法进行修改：

```java
public class Singleton {
    private static Singleton singleton = null;

    private Singleton() {
        System.out.println("生成了一个实例");
        slowdown();
    }

    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }

    private void slowdown() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}

```

为了确保生成多个实例，我们故意降低程序的处理速度，以确保每次调用都生成一个实例。

在以上代码中，如下条件判断是线程不安全的。

```java
if (singleton == null) {
	singleton = new Singleton();
}
```

在使用`singleton==null`判断第一个实例是否为null后，执行了下面语句。

```java
singleton = new Singleton();
```

但是，在赋值之前，其他线程可能会进行`singleton == null`判断。

因此，为了确保单例模式，在定义getInstance方法时，我们声明为synchronized方法后才是严谨的Singleton模式。

```java
public class Singleton {
    private static Singleton singleton = null;

    private Singleton() {
        System.out.println("生成了一个实例");
        slowdown();
    }

    public static synchronized Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }

    private void slowdown() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 相关设计模式

- [[08_Abstract_Fatctory模式]]
- [[07_Builder模式]]
- [[15_Facade模式]]
- [[06_Prototype模式]]

