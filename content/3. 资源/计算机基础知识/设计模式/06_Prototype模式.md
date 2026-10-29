---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Prototype模式

通常我们会使用以下方式生成Something类的实例。

```java
new something()
```

在Java中，我们可以使用new关键字指定类名来生成类的实例。像这样使用new来生成实例时，是必须指定类名的。在开发过程中，有时会有“在不指定类名的前提下生成实例”的需求。以下情况，我们不能根据类生成实例，而要根据现有的实例来生成实例。

- 对象种类繁多，无法将它们整合到一个类中
- 难以根据类生成实例时
- 想要解耦框架与生成的实例时

我们将要学习不根据类来生成实例，而是根据实例来生成新实例的Prototype模式（原型模式）。

在java语言中，我们可以使用clone创建出实例的副本。

## 例子

将字符串放入方框中显示出来或是加上上下划线显示出来。

类与接口如下表：

| 包        | 名字         | 说明                                                                 |
| --------- | ------------ | -------------------------------------------------------------------- |
| framework | Product      | 声明了抽象方法use和createClone接口                                   |
| framework | Manager      | 调用createClone方法复制实例的类                                      |
| 无名      | MessageBox   | 将字符串放入方框并且使其显示出来的类。实现了use方法和createClone方法 |
| 无名      | UnderlinePen | 给字符串加上下划线并使其显示出来的类。实现了use方法和createClone方法 |
| 无名      | Main         | 测试程序行为的类                                                     |

Product接口和Manager类属于framework包，负责复制实例。只要是实现了Product接口的类，调用它的createClone方法就可以复制出新的实例。

MessageBox类和UnderlinePen类这两个类都实现了Product接口。只要事先将这两个类“注册”到Manager类中，就可以随时复制新的实例。

![image-20221010104054981](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221010104054981-1665369656037-1.png)

Product接口是定义了复制功能的接口，实现了该接口的类的实例可以调用clone方法来自动复制实例。use方法是用于“使用”的方法。具体怎么“使用”，则被交给子类实现。createClone方法是用于复制实例的方法。

```java
public interface Product extends Clonable {
    public abstract  void use(String s);
    public abstract  Product crateClone();
}
```

Manager类使用Product接口来复制实例。

```java
public class Manager {
    private HashMap<String, Product> showcase = new HashMap<>();

    public void register(String name, Product proto) {
        showcase.put(name, proto);
    }

    public Product create(String protoname) {
        Product p  = (Product) showcase.get(protoname);
        return p.crateClone();
    }
}

```

**请注意**：在Product接口和Manager类的代码中完全没有出现MessageBox类和UnderlinePen类的名字，这也意味着我们可以独立地修改Product和Manager，不受MessageBox类和UnderlinePen类的影响。这点很重要，**一旦类中出现了别的类名，就意味着该类与其他类紧密地耦合在了一起**。Manager类中只有Product接口名，也就是说，Product接口成为了连接Manager类与其他类之间的桥梁。

MessageBox是将字符串放入方框并且使其显示出来的类。实现了use方法和createClone方法

```java
public class MessageBox implements Product {
    private char dochar;
    public MessageBox(char dochar) {
        this.dochar = dochar;
    }
    @Override
    public void use(String s) {
        int length = s.getBytes().length;
        for (int i = 0; i < length + 4; i++) {
            System.out.print(dochar);
        }
        System.out.println("");
        System.out.println(dochar + "" + s + "" + dochar);
        for (int i = 0; i < length + 4; i++) {
            System.out.print(dochar);
        }
        System.out.println("");
    }

    @Override
    public Product crateClone() {
        Product p = null;
        try {
            p = (Product) clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return p;
    }
}
```

UnderlinePen类的实现几乎相同，给字符串加上下划线并使其显示出来的类。实现了use方法和createClone方法

```java
public class UnderlinePen implements Product {
    private char ulchar;
    public UnderlinePen(char ulchar) {
        this.ulchar = ulchar;
    }
    @Override
    public void use(String s) {
        int length = s.getBytes().length;
        System.out.print("\"" + s + "\"");
        System.out.println("");
        for (int i = 0; i < length; i++) {
            System.out.print(ulchar);
        }
        System.out.println("");
    }

    @Override
    public Product crateClone() {
        Product p = null;
        try {
            p = (Product) clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return p;
    }
}
```

Main类首先生成了Manager的实例。接着，在Manager实例中注册了UnderlinePen的实例（带名字）和MessageBox的实例（带名字）。

```java
public class Main {
    public static void main(String[] args) {
        Manager manager = new Manager();
        UnderlinePen underlinePen = new UnderlinePen('~');
        MessageBox messageBox = new MessageBox('*');
        MessageBox sbox = new MessageBox('/');
        manager.register("strong message", underlinePen);
        manager.register("warning box", messageBox);
        manager.register("slash box", sbox);

        Product p1 = manager.create("strong message");
        p1.use("Hello, world");
        Product p2 = manager.create("warning box");
        p2.use("Hello,world");
        Product p3 = manager.create("slash box");
        p3.use("Hello, world");
    }
}
```

## 登场角色

- Prototype（原型）

  负责定义用于复制现有实例来生成新实例的方法。例子的Product接口扮演此角色。

- ConcretePrototype（具体的原型）

  负责实现复制现有实例并生成新实例的方法。例子中，MessageBox类和UnderlinePen类扮演此角色。

- Client（使用者）

  负责使用复制实例的方法生成新的实例。例子中，Manager类扮演此角色。

![image-20221010105912616](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221010105912616-1665370754025-3.png)

## 总结

### 不能根据类生成实例吗

一般情况下是可以的，但是可能会出现下面情况：

1. 对象种类繁多，无法将它们整合到一个类中时

   在示例程序中，出现了三种样式

   - 使用'~'为字符串添加下框线
   - 使用'*'为字符串添加边框
   - 使用'/'为字符串添加边框

   这里虽然只有3个样式，不过只要我们想做，不论多少样式都可以生成。如果每种样式都编写一个类，类的数量将会非常庞大，源程序的管理也会非常困难。

2. 难以根据类名生成实例时

   想想使用鼠标操作，每个动作都是一个实例，此时很难通过类名生成实例

3. 想要解耦框架与生成的实例时

   例子中，我们在Manager类的create方法中，没有使用类名，取而代之的是使用了“strong message”和“slash box”等字符串为生成实例命名。这种方式有更好的通用性

### 类名是束缚吗

在代码中出现类名不总是坏事。不过，**一旦在代码中出现要使用的类的名字，就无法与该类分离开来，也就无法实现复用。**

所谓复用，不包括修改源代码实现。我们需要保证即使没有这个java程序也能复用该类。

## 拓展：clone与Cloneable

### Java中的clone

Java语言为我们准备了用于复制实例的clone方法。要想调用clone方法，**被复制对象的类必须实现java.lang.Cloneable接口**，不论是被复制对象的类实现java.lang.Cloneable接口还是其某个父类实现Cloneable接口，亦或是被复制对象的类实现了Cloneable接口的子接口都可以。在示例程序中，MessageBox类和UnderlinePen类实现了Product接口，而Product接口则是Cloneable接口的子接口。

实现了Cloneable接口的类的实例可以调用clone方法进行复制，clone方法的返回值是复制出的新的实例(clone方法内部所进行的处理是分配与要复制的实例同样大小的内存空间，接着将要复制的实例中的字段的值复制到所分配的内存空间中去)。

如果没有实现Cloneable接口的类的实例调用了clone方法，则会在运行时抛出CloneNotSupportedException(不支持clone方法)异常。

总结如下。

- 实现了Cloneable接口的类的实例
  - 复制
- 没有实现Cloneable接口的实例
  - 发生CloneNotSupportedException异常

### clone在哪定义

clone定义在Object中

### clone方法进行的是浅复制

**clone方法只是将复制的实例直接复制到新的实例中**，换言之，没有考虑字段中所引用的实例的内容。例如，当字段保存的是数组时，如果是使用clone方法进行复制，则只会复制该数组的引用，并不会一一复制数组中的元素。

像上面这样的字段对字段（field-to-field-copy）的复制被称为浅复制（shallow copy）。

当使用clone方法无法满足时，可以重写clone方法，实现自己的复制（重写时，别忘了使用super.clone()来调用父类的clone方法）。

**clone方法只会进行复制，并不会调用示例中的构造函数。** 对于生成时需要进行特殊处理的类，需要自己实现clone方法，在其内部进行这些的初始化处理。

> 这里要回顾一下深拷贝与浅拷贝的区别（https://www.cnblogs.com/javastack/p/12759352.html）

## 相关设计模式

- [[Flyweight模式]]
- [[18_Memento模式]]
- [[11_Composite模式]]
- [[12_Decorator模式]]
- [[Command模式]]
