---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## 什么是模板

模板原意实质带有镂空文字的薄薄的塑料板。只要用笔在模板的镂空处进行临摹，即使手写也能写出整齐的文字。根据不同的笔可以写出不同的颜色。但无论用什么笔，到最后字的形状都是一样的。

## 什么是模板方法

Template Method模板是带有模板功能的模式，组成模板的方法被定义在父类中。方法都是抽象方法，所以查看父类不知怎么处理的，能知道的就是父类如何调用这些方法的。

实现上述的这些抽象方法就是子类。在子类中实现了抽象方法的具体处理。也就是说，不同的子类中实现不同的具体处理，当父类的模板方法被调用时程序行为也不同。但是，不论子类中的具体实现如何，处理的流程都会按照父类所定义的那样进行。

像这样**在父类中定义处理流程的框架，在子类中实现具体处理**的模式就称为Template Method模式。

## 例子

讲一段字符和字符串循环显示5次。

将会有AbstractDisplay、CharDisplay、StringDisplay、Main这4个类。

AbstractDisplay类中定义了display方法，其中分别调用了open、print、close这3个方法。虽然这3个方法已经在AbstractDisplay中被声明了，但都没有尸体的抽象方法。这里，调用抽象方法的display方法就是**模板方法**。

CharDisplay与StringDisplay类实现了open、print、close这3个抽象方法

Main用于测试。

![image-20221007104335768](./assets/image-20221007104335768-1665110616974-1.png)

AbstractDisplay类有4个方法，open、close、print都是抽象方法。display就是模板方法，实现了以下功能

- 开始打印
- 将字符或者字符串输出5次
- 结束打印

```java
public abstract class AbstractDisplay {
    public abstract void open();
    public abstract void close();
    public abstract void print();
    public final void display() {
        open();
        for (int i = 0; i < 5; i++) {
            print();
        }
        close();
    }
}
```

CharDisplay与StringDisplay是AbstractDisplay的子类，实现其中个性化的抽象类

```java
public class CharDisplay extends AbstractDisplay {
    private char ch;

    public CharDisplay(char ch) {
        this.ch = ch;
    }
    @Override
    public void open() {
        System.out.print("<<");
    }

    @Override
    public void close() {
        System.out.println(">>");
    }

    @Override
    public void print() {
        System.out.print(ch);
    }
}
```

```java
public class StringDisplay extends AbstractDisplay {
    private String string;
    private int width;

    public StringDisplay(String string) {
        this.string = string;
        this.width = string.length();
    }
    @Override
    public void open() {
        printLine();
    }

    @Override
    public void close() {
        printLine();
    }

    @Override
    public void print() {
        System.out.println("|" + string + "|");
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

最后，Main方法作为主程序进行调用

```java
public class Main {
    public static void main(String[] args) {
        CharDisplay charDisplay = new CharDisplay('H');
        StringDisplay stringDisplay = new StringDisplay("Hello World");
        charDisplay.display();
        stringDisplay.display();
    }
}
```

## 登场角色

- AbstractClass（抽象方法）

  负责模板方法，同时负责声明在模板方法声明的抽象方法。

- ConcreteClass（具体类）

  负责实现AbstractClass角色定义的抽象方法。

![image-20221007105249694](./assets/image-20221007105249694-1665111170668-1.png)

## 总结

### 可以使逻辑处理通用化

模板方法的好处：

1. 在父类的模板方法中编写了算法，因此无需在每个子类中再编写算法
2. 当在模板方法中出现bug时，只需要修改模板方法

### 父类子类之间的协作

此模式中，父类和子类是紧密联系、共同工作的。在子类中实现父类中声明的抽象方法时，必须要理解这些抽象方法被调用的时机。

### 父类子类的一致性

在示例程序中，不论是CharDispaly的实例还是StringDisplay的实例，都是先保存在AbstractDisplay类型的变量中，然后再调用display方法。

使用父类类型的变量保存子类实例的优点是：即使没有instance of 等指定子类的种类，程序也能正常工作。

**无论在父类类型的变量中保存哪个子类的实例，程序都可以正常运行，这种原则称为里氏原则**。里氏原则是通用的继承原则。

## 拓展：类的层次与抽象类

### 父类对子类的要求

理解类的层次是，通常是站在子类的角度进行思考的。也就是说，很容易着眼于以下几点。

- **在子类中可以使用父类中定义的方法**
- **可以通过在子类中增加方法以实现新功能**
- **在子类中重写父类的方法可以改变程序的行为**

换个角度，站在父类的角度。在父类中，声明了抽象方法，将该方法的实现交给了子类。换言之，就程序而言，生命抽象法师希望达到以下目的：

- **期待子类去实现抽象方法**
- **要求子类去实现抽象方法**

子类具有是现在父类中声明的抽象方法的责任。这种责任称为“子类责任”

### 抽象类的意义

这个话其实自己总结不好总结，并且不是很理解。

![image-20221007111841496](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221007111841496-1665112722631-3.png)

### 父类子类协作

父类子类相互协作撑起了整个程序。在父类放入过多，子类缺少灵活性，父类方法过少，子类臃肿。

在模板方法模式中，出来的流程被定义在父类中，具体的实现交给了子类。但是对于“如何划分处理级别，那些处理需要由父类完成，那些处理需要子类负责”并没有定义，需要程序开发者设计。

## 相关模式

- [[04_Factory_Method模式]]
- [[10_Strategy模式]]
