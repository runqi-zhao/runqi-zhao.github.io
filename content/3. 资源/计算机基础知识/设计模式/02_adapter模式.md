---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Adapter模式

举个例子：想让工作电压是直流12伏特的笔记本电脑在交流220伏特的AC电源下工作，应该怎么做？通常，使用AC适配器，将家庭用的交流220伏特电压转换成我们所需要的直流12伏特电压。这就是适配器(Adapter)的工作，**它位于实际情况与需求之间，填补两者之间的差异**。

在程序世界中，经常存在现有的程序无法直接使用，需要做适当的变换之后才能使用的情况。这种用于填补“现有的程序”和“所需的程序”之间差异的设计模式就是**Adapter模式**。

Adapter有以下两种

- **类适配器模式（使用继承的适配器）**
- **对象适配器模式（使用委托的适配器）**

下面将通过例子说明。

## 例子

### 类适配器模式

示例程序将会把一段输入的字符串显示为(Hello)或是`*Hello*`的简单程序。

目前在Banner类中，有将字符串用括号括起来的showWithParen的方法，和将字符串用`*`号括起来的showWithAster方法。假设这个Banner类是类似前文中“交流220伏特电压”的“实际情况”。

假设Print接口中声明了两种方法，及弱化字符串显示（加括号）的printWeak(weak有弱化的意思)方法，和强调字符串显示（加`*`号）的printStrong(Strong有强化的意思)方法。假设这个接口类似与前文中"直流12伏特电压"的“需求”。

现在要做的事就是使用Banner类编写一个实现的Print接口的类，也就是说要做一个将“交流220伏特电压”转换成“直流12伏特电压”的适配器。

扮演适配器角色是PrintBanner类。该类继承了Banner类并实现了“需求”-Print接口。PrintBanner类使用showWithParen方法实现了printWeak，使用showWithAster方法实现了printStrong。这样，PrintBanner类就具有了适配器的功能，电源的比喻和示例程序的对应关系如下：

|          | 电源的比喻  | 示例程序                                 |
| -------- | ----------- | ---------------------------------------- |
| 实际情况 | 交流220伏特 | Banner类（showWithParen、showWithAster） |
| 变换装置 | 适配器      | StringBanner类                           |
| 需求     | 直流12伏特  | Print接口（printWeak、printStrong）      |

![image-20221006101503727](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221006101503727-1665022505047-1.png)

先看实际情况的Banner类

```java
public class Banner {
    private String string;
    public Banner(String string) {
        this.string = string;
    }

    public void showWithParen() {
        System.out.println("(" + string + ")");
    }

    public void showWithAster() {
        System.out.println("*" + string + "*");
    }
}
```

再看Print接口，使用的是“需求”接口。

```java
public interface Print {
    public abstract void printWeak();
    public abstract void printStrong();
}
```

接下来是PrintBanner类扮演适配器的角色。继承了Banner类，继承了showWithParen方法和showWithAster方法。同时，它又实现(implements)了Print接口，实现了printWeak方法和printStrong方法。

```java
public class PrintBanner extends Banner implements Print {

    public PrintBanner(String string) {
        super(string);
    }

    @Override
    public void printWeak() {
        showWithParen();
    }

    @Override
    public void printStrong() {
        showWithAster();
    }
}
```

Main类，通过适配器来达到输出。

```java
public class Main {
    public static void main(String[] args) {
        Print p = new PrintBanner("Hello");
        p.printWeak();
        p.printStrong();
    }
}
```

这里我们将PrintBanner类的实例保存在了Print类型的变量中。在Main类中，我们是使用了Print接口（即调用printWeak方法和printStrong）来编程。对于Main而言，Banner类中的方法完全被隐藏。

Main类并不知道PrintBanner类是如何实现的，这样就可以在不用对Main类进行修改的情况下改变PrintBanner类的具体实现了。

### 委托的适配器

> 关于委托
>
> ”委托“通俗点说就是“交给别人”。比如，当我们无法出席重要会议时，可以写一份委任书，说明“我无法出席，安排小赵替我出席”。委托的意思是一样的。在Java语言中，委托就是指将某个方法中的实际处理交给其他的实例方法。

Main类与Banner类与前面的例子完全相同，不过这里Print不是接口而是类。

也就是说，打算利用一个Banner类实现一个类，该类的方法和Print类的方法相同。由于Java中无法同时继承两个类，因此无法将PrintBanner类分别定义为Print类和Banner的子类。因此我们在PrintBanner类中创建Banner的实例。吃我们使用这个字段来调用代码。

这样就行了一种委托关系。当PrintBanner类的printWeak被调用的时候，并不是PrintBanner类自己进行处理，而是将处理交给了其他实例（Banner实例）的showWithParen方法。

![image-20221006103824726](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221006103824726-1665023905958-3.png)

print类

```java
public abstract class Print {
    public abstract void Print();
    public abstract void PrintStrong();
}
```

PrintBanner类

```java
public class PrintBanner extends Print {
    private Banner banner;

    @Override
    public void Print() {
        banner.showWithParen();
    }

    @Override
    public void PrintStrong() {
        banner.showWithAster();
    }
}
```

## Adapter角色

- Target（对象）

  负责定义所需的方法（12V直流电压）

- Client（请求者）

  负责使用Target角色定义的方法进行处理（12V驱动电脑）

- Adaptee（被适配）

  已有的条件（100V电压）

- Adapter（适配）

  使用Adaptee角儿的方法来满足Target角色的需求。（转换）

在类适配器模式中，Adapter角色通过继承来使用Adaptee角色，而在对象适配器模式中，Adapter角色通过委托来使用Adaptee角色。

类适配器类图：

![image-20221006104714579](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221006104714579-1665024436057-5.png)

委托适配器类图：![image-20221006104739120](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221006104739120-1665024460627-7.png)

## 总结

### 何时使用

当有了一个现成代码的时候，我们需要使用时进行一些简单的修改，就可以使用

### 没有现成代码

使用Adapter模式可以在完全不改变现有代码的前提下使现有代码是培育新的接口（API）。此外，在Adapter模式中，并非一定需要现成的代码，只要知道现有类的功能，就可以编写出新的类。

### 版本与兼容

当版本进行升级时，我们可以写一个适配器，将无法抛弃的旧版本进行适配。

![image-20221006105253644](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221006105253644-1665024775157-9.png)

### 功能完全不同的类

当功能不同时，不能使用适配器模式，例如，我们无法将220V电压转变为水流流出。

## 相关设计模式

- [[09_Bridge模式]]
- [[12_Decorator模式]]

