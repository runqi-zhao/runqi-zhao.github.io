---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Bridege模式

将类的功能层次结构与实现层次结构分离

### 类的层次结构

类的层次结构有两个作用

#### 希望增加新功能时

假设现在有一个类Something。当我们想在Something中增加新功能时（想要增加一个具体的方法时），会编写一个Something的子类（派生类），即SomethingGood类，这样，就形成了一个小小的层次结构。

```
Something
    |
    |__SomrthingGood
```

这就是为了增加新功能而产生的层次结构

- **父类具有基本功能**
- **在子类中增加新的功能**

以上这种结构称为“**类的功能层次结构**”。

如果想要继续在SomethingGood类的基础上增加新的功能，此时我们可以同样编写一个SomethingGood的子类，即SomethingBetter类。此时，类的层次结构就加深了。

> 注意：通常来说，类的层次结构关系不应该过深

### 类的实现层结构

#### 希望增加新的实现时

在[[03_Template_Method模式]]中，介绍了抽象类的作用。抽象类声明了一些抽象方法，定义了接口（API），然后子类负责去实现这些抽象方法。父类的任务是通过声明抽象方法的方式定义接口（API），而子类的任务是实现抽象方法。这是这种分担模式，才可以编写出具有高可替代性的类。

例如，当子类ConcreteClass实现父类AbstractClass类的抽象方法是，他们之间就构成了一个小小的层次结构。

```
AbstractClass
    |
    |__ ConcreteClass
```

这里的类的层次结构并非用于增加功能，也就是说，这种层次结构并非用于方便我们增加新的方法。他的真正作用是帮助我们实现下面这样的任务分担。

- **父类通过声明抽象方法来定义接口（API）**
- **子类通过实现具体方法来实现接口（API）**

这种层次结构被称为”**类的层次结构**“。

## 类的层次结构的混杂与分离

将类的”类的功能层次结构“与”类的实现层次结构“**分离为两个独立的类层次结构**。这样在之后无论是需要增加新功能还是新层次时，只需要对其中一个修改，就能实现功能。

桥接模式就相当于两者兼得联系。

## 例子

程序是”显示一些东西“。乍一听抽象，仔细向下看

| 在桥的哪一侧     | 名字              | 说明                               |
| ---------------- | ----------------- | ---------------------------------- |
| 类的功能层次结构 | Display           | 负责”显示“的类                     |
| 类的功能层次结构 | CountDisplay      | 增加了”只显示规定次数“这一功能的类 |
| 类的实现层次结构 | DisplayImpl       | 负责”显示“的类                     |
| 类的实现层次结构 | StringDisplayImpl | ”用字符串显示“的类                 |
|                  | Main              | 测试程序的类                       |

![image-20221013101945414](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221013101945414-1665627586752-1.png)

Diplay类的功能是抽象的，负责”显示一些东西“。该类位于”类的功能层次结构“的最上层。

在impl字段中保存的是实现了Display类的具体功能的实例。

该实例通过Display类的构造函数被传递给Display类，然后保存在impl字段中，一共后面的处理使用。

open、print、close是Display类提供的接口，它们表示”显示的步骤“：

- open是显示前的处理
- print是显示处理
- close是显示后的处理

这3个方法都调用了impl字段的实现方法。这样，Display的接口就被转换成为了DisplayImpl的接口（API）。

```java
public class Display {
    private DisplayImpl impl;
    public Display(DisplayImpl impl) {
        this.impl = impl;
    }

    public void open() {
        impl.rawOpen();
    }

    public void print() {
        impl.rawPrint();
    }

    public void close() {
        impl.rawClose();
    }

    public void display() {
        open();
        print();
        close();
    }
}
```

CountDisplay是在Display的基础上增加了一个新功能。也就是”类的功能层次结构“。

```java
public class CountDisplay extends Display{
    public CountDisplay(DisplayImpl impl) {
        super(impl);
    }

    public void multiDisplay(int times) {
        open();
        for (int i = 0; i < times; i++) {
            print();
        }
        close();
    }
}
```

DisplayImpl是”类的实现层次结构“的最上层。

```java
public abstract class DisplayImpl {
    public abstract void rawOpen();
    public abstract void rawPrint();
    public abstract void rawClose();
}
```

StringDisplayImpl类是显示字符串的类

```java
public class StringDisplayImpl extends DisplayImpl{
    private String string;
    private int width;

    public StringDisplayImpl(String string) {
        this.string = string;
        this.width = string.getBytes().length;
    }

    @Override
    public void rawOpen() {
        printLine();
    }

    @Override
    public void rawPrint() {
        System.out.println("|" + string + "|");
    }

    @Override
    public void rawClose() {
        printLine();
    }

    private void printLine() {
        System.out.print("+");
        for (int i = 0; i < width; i++) {
            System.out.print("-");
        }
        System.out.println("+");
    }
}
```

DisplayImpl和StringDisplayImpl这两个类相当于”类的实现层次结构“。

Main类是进行测试

```java
public class Main {
    public static void main(String[] args) {
        Display d1 = new Display(new StringDisplayImpl("Hello, China."));
        Display d2 = new CountDisplay(new StringDisplayImpl("Hello, World."));
        CountDisplay d3 = new CountDisplay(new StringDisplayImpl("Hello, Universe."));
        d1.display();
        d2.display();
        d3.display();
        d3.multiDisplay(5);
    }
}
```

## 登场角色

- Abstraction（抽象化）

  位于”类的功能层次结构“的最上层。使用Implementor角色的方法定义了基本的功能。该角色中保存了Implementor的实例。

- RefinedAbstraction（改善后的抽象化）

  在Abstraction角色的基础上增加了一个新功能的角色。

- Implementor（实现者）

  位于”类的实现层次结构“的最上层。定义了用于实现Abstraction角色的接口（API）的方法。

- ConcreteImplementor（具体实现者）

  负责在Implementor角色中定义的接口。

![image-20221013103644082](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221013103644082-1665628605491-3.png)

## 总结

### 分开之后更容易扩展

Bridge模式是将”类的功能层次结构“与”类的实现层次结构“分离开。将类的这两个层次结构分离开有利于独立的对他们进行拓展。

当想要增加功能时，只需要对”类的功能层次结构“增加类即可，不必对”类的实现层次结构”做任何修改。**增加后的功能可以被“所有的实现”使用。**

### 继承是强关联，委托是弱关联

继承虽然很容易拓展，但是类之间会形成一种强关联关系（只要不修改代码，就无法改变这种关系）。

委托相当于不是自己工作，而是将工作交给别人。例如，上面李忠Display类并非自己工作，而是将工作“交给impl”。只有Display类的实例生成时，才与作为参数被传入的类构成关联。

## 相关设计模式

- [[03_Template_Method模式]]
- [[08_Abstract_Fatctory模式]]
- [[02_adapter模式]]
