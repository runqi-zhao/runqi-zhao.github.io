---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
想象一个文件系统，在这文件系统中，有文件夹的概念，文件夹里面可以有文件，也可以有子文件夹，可以说，文件夹形成了一种容器结构、递归结构。

文件夹与文件虽然属于不同类型的对象，但是它们都“可以被放入到文件夹中”。文件夹和文件有时也被称为“目录条目”。在目录条目中，文件夹和文件当作一种对象看待。（即一致性）。

**能够使容器与内容具有一致性，创造出递归结构**的模式就是Composite模式。

## 例子

列出文件和文件夹的一览。

| 名字                    | 说明                                          |
| ----------------------- | --------------------------------------------- |
| Entry                   | 抽象类，用来是实现File类和Directory类的一致性 |
| File                    | 表示文件的类                                  |
| Directory               | 表示文件夹的类                                |
| FileTreatementException | 表示想文件中增加Entry时发生的异常类           |
| Main                    | 测试程序行为的类                              |

![image-20221015094233155](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221015094233155-1665798154192-1.png)

Entry类是一个表示目录条目的抽象类。

```java
public abstract class Entry {
    /**
     * 获取名字
     * @return name
     */
    public abstract String getName();

    /**
     * 获取大小
     * @return size
     */
    public abstract int getSize();

    /**
     * 加入条目
     * @param entry 条目
     * @return 加入的条目
     * @throws FileTreatmentException
     */
    public Entry add(Entry entry) throws FileTreatmentException {
        throw new FileTreatmentException();
    }

    public void printList() {
        printList("");
    }

    /**
     * 为一览加上前缀并显示条目一览,显示代表类的文字
     * @param prefix 前缀
     */
    protected abstract void printList(String prefix);

    @Override
    public String toString() {
        return getName() + "(" + getSize() + ")";
    }
}
```

File类时表示文件类，时Entry的子类

```java
public class File extends Entry{
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
    protected void printList(String prefix) {
        System.out.println(prefix + "/" + this);
    }
}
```

Directory类是表示文件夹的类。也是Entry的子类。

getName方法只是简单地返回了name,但在getsize方法中则进行了计算处理。它会遍历directory字段中的所有元素，然后计算出它们的大小的总和。请注意以下语句。

```java
size +entry.getsize();
```

这里，在变量size中加上了entry的大小，但entry可能是File类的实例，也可能是Directory类的实例。不过，不论它是哪个类的实例，我都可以通过getsize方法得到它的大小。

这就是Composite模式的特征容器与内容的一致性的表现。不管entry究竟是File类的实例还是Directory类的实例，它都是Entry类的子类的实例，因此可以放心地调用getSize方法。即使将来编写了其他Entry类的子类，它也会实现getsize方法，因此Directory类的这部分代码无需做任何修改。

如果entry是Directory类的实例，调用entry.getSize()时会将该文件夹下的所有目录条目的大小加起来。如果其中还有子文件夹，又会调用子文件夹的getsize方法，形成递归调用。这样一来，大家应该能够看出来，(getsize方法的递归调用与Composite模式的结构是相对应的。

add方法用于向文件夹中加人文件和子文件夹。该方法并不会判断接收到的entry到底是Directory类的实例还是File类的实例，而是通过如下语句直接将目录条目加入至directory字段中。“加入”的具体处理则被委托给了ArrayList类。

```java
directory.add(entry)
```

printList方法用于显示文件夹的目录条目一览。printList方法也会递归调用，这一点和getsize方法一样。而且，printList方法也没有判断变量entry究竟是File类的实例还是Directory类的实例，这一点也与getSize方法一样。这是因为容器和内容具有一致性。

```java
public class Directory extends Entry{
    private String name;
    private ArrayList directory = new ArrayList();

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
        Iterator it = directory.iterator();
        while (it.hasNext()) {
            Entry entry = (Entry) it.next();
            size += entry.getSize();
        }
        return size;
    }

    public Entry add(Entry entry) {
        directory.add(entry);
        return this;
    }

    @Override
    protected void printList(String prefix) {
        System.out.println(prefix + "/" + this);
        Iterator it = directory.iterator();
        while (it.hasNext()) {
            Entry entry = (Entry) it.next();
            entry.printList(prefix + "/" + name);
        }
    }
}
```

FileTreatMentExpection：异常类，文件调用add方法时抛出。

```java
public class FileTreatmentException extends RuntimeException{
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
            bindir.add(new File("vi", 1000));
            bindir.add(new File("latex", 20000));
            rootdir.printList();

            System.out.println("");
            System.out.println("Making user entries...");
            Directory yuki = new Directory("yuki");
            Directory hanako = new Directory("hanako");
            Directory tomura = new Directory("tomura");
            usrdir.add(yuki);
            usrdir.add(hanako);
            usrdir.add(tomura);
            yuki.add(new File("diary.html", 100));
            yuki.add(new File("Composite.java", 200));
            hanako.add(new File("memo.tex", 300));
            tomura.add(new File("game.doc", 400));
            tomura.add(new File("junk.mail", 500));
            rootdir.printList();
        } catch (FileTreatmentException e) {
            e.printStackTrace();
        }
    }
}

```

## 登场角色

- Leaf（树叶）

  表示“内容”的角色

- Composite（复合物）

  表示容器的角色。可以在其中放入Leaf角色和Composite角色具有一致性的角色

- Component

  是Leaf角色和Composite角色具有一致性的角色。

- Client

  使用Composite角色。

将Composite角色与它的内部Component角色（即Leaf角色或Composite角色）看成父亲与孩子的关系，getChild方法的作用是从Component角色获取这些“孩子们”。

![image-20221015095651972](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221015095651972-1665799013011-3.png)

## 总结

### 多个和单个的一致性

使用Composite模式可以使容器与内容具有一致性，也可以称其为**多个和单个的一致性**
即将多个对象结合在一起，当作一个对象进行处理。

例如，让我们试想一下测试程序行为时的场景。现在假设Test1是用来测试输入数据来自键盘输入时的程序的行为，Test2是用来测试输入数据来自文件时的程序的行为，Test3是用来测试输入数据来自网络时的程序的行为。如果我们想将这3种测试统一为“输人测试”，那么Composite模式就有用武之地了。我们可以将这几个测试结合在一起作为“输入测试”，或是将其他几个测试结合在一起作为“输出测试”，甚至可以最后将“输入测试”和“输出测试”结合在一起作为“输入输出测试”。

### Add方法应该放在哪里

在示例程序中，Entry类中定义了add方法，所做的处理是抛出异常，这是因为能使用add方法的只能是Directory类。下面我们学习一下各种add方法的定义位置和实现方法。

#### 方法1：定义在Entry类中，报错

将add方法定义在Entry类中，让其报错，这是示例程序中的做法。能使用add方法的只有Directory类，它会重写add方法，根据需求实现其处理。

File类会继承Entry类的add方法，虽然也可以调用它的add方法，不过会抛出异常。

#### 方法2：定义在Enty类中，但什么都不做

也可以将add方法定义在Entry类中，但什么处理都不做。

#### 方法3：声明在Entry类中，但不实现

也可以在Entry类中声明add抽象方法。如果子类需要add方法就根据需求实现该方法，如果不需要add方法，则可以简单地报错。该方法的优点是所有子类必须都实现add方法，不需要add方法时的处理也可以交给子类自己去做决定。不过，使用这种实现方法时，在Fi1e一方中也必须定义本来完全不需要的add(有时还包括remove和getChild)方法。

#### 方法4：只定义在Directory类中

因为只有Directory类可以使用add方法，所以可以不在Entry类中定义add方法，而是只将其定义在Directory类中。不过，使用这种方法时，如果要向Entry类型的变量（实际保存的是Directory类的实例)中add时，需要先将它们一个一个地类型转换(cast)为Directory
类型。

### 到处都存在递归结构

在示例程序中，我们以文件夹的结构为例进行了学习，但实际上在程序世界中，到处都存在递归结构和Composite模式。例如，在视窗系统中，一个窗口可以含有一个子窗口，这就是Composite模式的典型应用。此外，在文章的列表中，各列表之间可以相互嵌套，这也是一种递归
结构。将多条计算机命令合并为一条宏命令时，如果使用递归结构实现宏命令，那么还可以编写出宏命令的宏命令。另外，通常来说，树结构的数据结构都适用Composite模式。

## 相关设计模式

- [[Command模式]]
- [[13_Visitor模式]]
- [[12_Decorator模式]]
