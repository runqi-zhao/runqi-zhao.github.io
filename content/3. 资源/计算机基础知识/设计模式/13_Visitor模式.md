---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Visitor模式

在Visitor模式中，数据结构预处理分离开来。

## 例子

用[[11_Composite模式]]中用到的那个文件和文件夹的例子作为访问者要访问的数据结构。访问者和访问由文件和文件夹构成的而数据结构，然后显示出文件和文件夹的一览。

| 名字                    | 说明                                                       |
| ----------------------- | ---------------------------------------------------------- |
| Visitor                 | 表示访问者的抽象类，他访问文件和文件夹                     |
| Element                 | 表示数据结构的接口，他接受访问者的访问                     |
| ListVisitor             | Visotor类的子类，显示文件和文件夹一览                      |
| Entry                   | File类和Directory类的父类，它是抽象类（实现了Element接口） |
| File                    | 表示文件的类                                               |
| Directory               | 表示文件夹的类                                             |
| FileTreatementException | 表示文件中add时发生的异常的类                              |
| Main                    | 测试程序行为的类                                           |

![image-20221017102909546](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221017102909546.png)

Visitor类时表示访问者的类。访问者依赖于她所访问的数据结构（即File类和Directory类）。

```java
public abstract class Visitor {
    public abstract void visit(File file);
    public abstract void visit(Directory directory);
}
```

Element接口时接受访问者的访问的接口。

```java
public interface Element {
    public abstract void accept(Visitor v);
}
```

虽然Entry类在本质上与Composite模式中的Entry类是一样的，不过本章中的Entry类实现（implements)了Element接口。这是为了让Entry类适用于Visitor模式。实际上实现Element接口中声明的抽象方法accept的是Entry类的子类—
File类和Directory类。

add 方法仅对 Directory类有效，因此在Entry类中，我们让它简单地报错。同样地，用于获取 Iterator的iterator 方法也仅对 Directory类有效，我们也让它简单地报错。

```java
public abstract class Entry implements Element {
    public abstract String getName();
    public abstract int getSize();
    public Entry add(Entry entry) throws FileTreatmentException {
        throw new FileTreatmentException();
    }

    public Iterator iterator() throws FileTreatmentException {
        throw new FileTreatmentException();
    }

    public String toString() {
        return getName() + "(" + getSize() + ")";
    }
}
```

File类与Composite模式中的File类是一样。注意理解accept接口。

```java
public class File extends Entry {
    private String name;
    private int size;
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    @Override
    public String getName() {
        return name; 
    }

    @Override
    public int getSize() {
        return size;
    }

    @Override
    public void accept(Visitor v) {
        v.visit(this);
    }


}
```

Directory类是表示文件夹的方法。与Composite模式中的Directory类相比，本章中的Directory类中增加了下面两个方法。

第一个方法是 iterator 方法。iterator方法会返回Iterator，我们可以使用它遍历文件夹中的所有目录条目（文件和文件夹）。

第二个方法当然就是accept方法了。与File类中的accept方法调用了visit(File)方法一样，Directory类中的 accept 方法调用了 visit（Directory）方法。这样就可以告诉访问者“当前正在访问的是Directory类的实例”。

```java
public class Directory extends Entry {
    private String name;
    private ArrayList<Entry> dir = new ArrayList<>();
    public Directory(String name) {
        this.name = name;
    }
    @Override
    public String getName() {
        return name;
    }

    @Override
    public int getSize() {
        int size = 0;
        Iterator<Entry> it = dir.iterator();
        while (it.hasNext()) {
            Entry entry = it.next();
            size += entry.getSize();
        }
        return size;
    }

    @Override
    public Entry add(Entry entry) throws FileTreatmentException {
        dir.add(entry);
        return this;
    }

    @Override
    public Iterator iterator() {
        return dir.iterator();
    }

    @Override
    public void accept(Visitor v) {
        v.visit(this);
    }
}
```

ListVisitor类是Visitor类的子类，它的作用是访问数据结构并显示元素的一览。因为ListVisitor类是Visitor类的子类，所以它实现了visit(File）方法和visit(Directory）方法。

currentdir字段中保存的是现在正在访问的文件夹名字。visit（File）方法在访问者访问文件时会被File类的accept 方法调用，参数 file是所访问的File类的实例。也就是说.visit(File）方法是用来实现“**对File类的实例要进行的处理**”的。在本例中，我们实现的处
理是先显示当前文件夹的名字(currentdir)，然后显示间隔符号"/"，最后显示文件名。visit(Directory)方法在访问者访问文件夹时会被Directory类的accept方法调用，参数 directory是所访问的Directory类的实例。

在visit(Directory）方法中实现了“**对Directory类的实例要进行的处理**”。

本例中我们是如何实现的呢? 与visit（File）方法一样，我们先显示当前文件夹的名字，接着调用 iterator 方法获取文件夹的工terator，然后通过 Iterator遍历文件夹中的所有目录条目并调用它们各自的accept方法。由于文件夹中可能存在着许多目录条目，逐一访问会非常困难。accept方法调用visit方法,visit方法又会调用accept方法，这样就形成」非常复杂的递归调用。通常的递归调用是某个方法调用自身，在Visitor模式中，则是**accept方法与visit方法之间相互递归调用**。

```java
public class ListVisitor extends Visitor {
    private String currentdir = "";

    @Override
    public void visit(File file) {
        System.out.println(currentdir + "/" + file);
    }

    @Override
    public void visit(Directory directory) {
        System.out.println(currentdir + "/" + directory);
        String savedir = currentdir;
        currentdir = currentdir + "/" + directory.getName();
        Iterator it = directory.iterator();
        while (it.hasNext()) {
            Entry entry = (Entry) it.next();
            entry.accept(this);
        }
        currentdir = savedir;
    }

}
```

FileTreatmentException异常类

```java
public class FileTreatmentException extends RuntimeException {
    public FileTreatmentException() {

    }
    public FileTreatmentException(String msg) {
        super(msg);
    }
}
```

Main类进行测试

```java
public class Main {
    public static void main(String[] args) {
        try {
            System.out.println("Making root entries...");
            Directory rootdir = new Directory("root");
            Directory bindir = new Directory("bin");
            Directory tmpdir = new Directory("tmp");
            Directory usrdir = new Directory("usr");
            rootdir.add(bindir);
            rootdir.add(tmpdir);
            rootdir.add(usrdir);
            bindir.add(new File("vi", 10000));
            bindir.add(new File("latex", 20000));
            rootdir.accept(new ListVisitor());

            System.out.println("");
            System.out.println("Making use entries...");
            Directory yuki = new Directory("yuki");
            Directory hanko = new Directory("hanko");
            Directory tomura = new Directory("tomura");
            usrdir.add(yuki);
            usrdir.add(hanko);
            usrdir.add(tomura);
            yuki.add(new File("diary.html", 100));
            yuki.add(new File("Composite.java", 200));
            hanko.add(new File("memo.tex",300));
            tomura.add(new File("game.doc", 400));
            tomura.add(new File("junk.mail", 500));
            rootdir.accept(new ListVisitor());
        } catch (FileTreatmentException e) {
            e.printStackTrace();
        }
    }
}
```

### Visotr与Element之间的相互调用

这里我们再结合时序图来学习一下示例程序的处理流程

为了方便理解，我们在图中展示了当<font color="red">一个文件夹下有两个文件时</font>，示例程序的处理流程。

1. 首先，Main类生成ListVisitor的实例。在示例程序中，Main类还生成了其他的Directory类和File类的实例，但在本图中我们省略了。
2. 接着，Main类调用 Directory 类的accept方法。这时传递的参数是ListVisitor的实例，但我们在本图中省略了。
3. Directory类的实例调用接收到的参数 ListVisitor 的visit（Directory)方法
4. 接下来，ListVisitor类的实例会访问文件夹，并调用找到的第一个文件的accept 方法。传递的参数是自身(this）。
5. File的实例调用接收到的参数 ListVisitor的visit（File）方法。请注意，这时ListVisitor的visit（Directory).还在执行中(并非多线程执行，而是表示visit(Directory)还存在于调用堆栈(callstack）中的意思。在时序图中，表示生命周期的长方形的右侧发生了重叠就说明了这一点）。
6. 从Visit(File) 返回到 accept，接看义从acept☑返回出来，然后调用另外一个File的实例（同一文件夹下的第二个文件）的accept 方法。传递的参数是 ListVisitor的实例 this。
7. 与前面一样，File的实例调用visit(File）方法。所有的处理完成后，逐步返回，最后回到Main类中的调用 accept 方法的地方。

![image-20221017105619869](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221017105619869-1665975381585-1.png)

在阅读时序图时，请大家注意以下几点。

- 对**于 Directory类的实例和 File 类的实例，我们调用了它们的accept 方法**
- **对于每一个Directory类的实例和 File 类的实例，我们只调用了一次它们的accept 方法**
- **对于 ListVisitor 的实例，我们调用了它的visit(Directory) 和visit(File)方法**
- **处理visit(Directory) 和 visit(File)的是同一个ListVisitor的实例**

## 登场角色

### Visitor（访问者）

负责对数据结构中每个具体的元素，声明一个用于访问XXXX的visit(XXXX)方法。visit(XXXX)适用于处理XXXX的方法，负责实现该方法的是ConcreteVisitor角色。

### ConcreteVisitor

负责实现Visitor角色所定义的接口(API)。实现所有的visit(XXXX)方法，及实现如何处理每个ConcreteElement角色。

### Element

表示Visitor角色的访问对象。声明了接受访问这的accept方法。

### ConcreteElement

负责实现Element角色所定义的接口

### ObjectStructure

负责处理Element角色的集合。ConcreteVisitor角色为每个Element角色都准备了处理方法。在示例程序中，由Directory类扮演此角色（分饰两角）。

![image-20221017110521784](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221017110521784-1665975923123-3.png)

## 总结

### 双重分发

我们来整理一下 Visitor模式中方法的调用关系吧。

accept（接受）方法的调用方式如下。

```java
element.accept(visitor);
```

而visit（访问）方法的调用方式如下。

```
visitor.visit(element);
```

对比一下这两个方法会发现，它们是相反的关系。element 接受visitor，而visitor又访问element。

在Visitor模式中, ConcreteElement和ConcreteVisitor这两个角色共同决定了实际进行的处理。这种消息分发的方式一般被称为**双重分发**（doubledispatch）。

### 为何搞复杂

Visitor模式的目的就是奖处理从数据结构中分离出来。数据结构很重要，它能将元素集合和关联在一起。但是，需要注意的是，保存数据结构与以数据结构为基础进行处理是两种不同的东西。

在示例程序中，我们创建了 ListVisitor类作为显示文件夹内容的ConcreteVisitor角色。通常，ConcreteVisitor角色的开发可以种方干 File类和Directory类。Visitor模式提高了File类和Directory类作为组件的独立性。如果将进行处理的方法定义在File类和Directory类中，当每次要扩展功能 增加新的“外理”时 就不得不修改File类和Directoy类。

### 开闭原则-对扩展开放，对修改关闭

<font color="red">在不修改现有代码的前提下进行扩展</font>，这就是开闭原则。

> 提问：为什么String类不允许被继承，是否违反了开闭原则，这么做的原因是什么？
>
> 答：<font color="red">是效率原因</font>。
> String类是Java语言中用于处理字符串的基本类，扮演着非常重要的角色。因此，**Java编译器以“不能继承String类”为前提对String类的处理速度和内存消耗量都进行了优化**。

### 易于增加ConcreteVisitor角色

### 难以增加ContereElement角色

增加时，我们选哟在Visitor类以及对应的子类都进行修改

### Visitor工作所需条件

Element角色必须想Visitor角色公开足够多的条件。例如，在例子中，visit方法需要使用目录条目的eaccept方法，为此，Directory类必须提供用于获取每个目录条目的iterator方法。

但是如果公开了不应当被公开的信息，将来需要修改数据结构是就会变得很困难。

## 相关设计模式

- [[01_Iterator模式]]
- [[11_Composite模式]]
- [[Interpreter模式]]
