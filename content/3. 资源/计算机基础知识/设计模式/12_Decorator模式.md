---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

假如现在有一块蛋糕，如果只涂上奶油，其他什么都不加，就是奶油蛋糕。如果加上草莓，就是草莓奶油蛋糕。如果再加上一块黑色巧克力板，上面用白色巧克力写上姓名，然后插上代表年龄的蜡烛，就变成了一块生日蛋糕。

不论是蛋糕、奶油蛋糕、草莓蛋糕还是生日蛋糕，它们的核心都是蛋糕。不过，经过涂上奶油，加上草莓等装饰后，蛋糕的味道变得更加甜美了，目的也变得更加明确了。

程序中的对象与蛋糕十分相似。首先有一个相当于蛋糕的对象，然后像不断地装饰蛋糕一样地不断地对其增加功能，它就变成了使用目的更加明确的对象。
**像这样不断地为对象添加装饰的设计模式被称为Decorator模式**。Decorator指的是“装饰物”。本章中，我们将学习Decorator模式的相关知识。

## 例子

给文字添加边框

| 名字          | 说明                     |
| ------------- | ------------------------ |
| Display       | 用于显示字符串的抽象类   |
| StringDisplay | 用于显示单行字符串的类   |
| Border        | 用于显示装饰边框的抽象类 |
| SideBorder    | 用于只显示左右边框的类   |
| FullBodrder   | 用于显示上下左右边框的类 |
| Main          | 测试程序                 |

![image-20221016101946932](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221016101946932-1665886787928-1.png)

Display是可以显示多行字串的抽象类

```java
public abstract class Display {
    public abstract int getColumns();
    public abstract int getRows();
    public abstract String getRowText(int row);
    public final void show() {
        for (int i = 0; i < getRows(); i++) {
            System.out.println(getRowText(i));
        }
    }
}
```

StringDisplay是用于显示单行字符串的类

```java
public class StringDisplay extends Display {
    private String string;

    public StringDisplay(String string) {
        this.string = string;
    }

    @Override
    public int getColumns() {
        return string.getBytes().length;
    }

    @Override
    public int getRows() {
        return 1;
    }

    @Override
    public String getRowText(int row) {
        if (row == 0) {
            return string;
        } else {
            return null;
        }
    }
}
```

Border类是装饰边框的抽象类。虽然它所表示的是装饰边框，但它也是Display类的子类。

也就是说，通过继承，装饰边框与被装饰物具有了相同的方法。具体而言，Border类继承了父类的 getColumns、getRows、getRowText、show等各方法。从接口（API）角度而言，装饰边框（Border )与被装饰物(Display）具有相同的方法也就意味着它们具有**一致性**。

```java
public abstract class Border extends Display {
    public Display display;
    public Border(Display display) {
        this.display = display;
    }
}
```

SiderBorder类用于横向获取字符

```java
public class SideBorder extends Border {
    private char broderChar;

    public SideBorder(Display display, char broderChar) {
        super(display);
        this.broderChar = broderChar;
    }

    @Override
    public int getColumns() {
        return 1 + display.getColumns() + 1;
    }

    @Override
    public int getRows() {
        return display.getRows();
    }

    @Override
    public String getRowText(int row) {
        return broderChar + display.getRowText(row) + broderChar;
    }
}
```

FullBorder类是上下左右都增加边框

```java
public class FullBorder extends Border {
    public FullBorder(Display display) {
        super(display);
    }

    @Override
    public int getColumns() {
        return 1 + display.getColumns() + 1;
    }

    @Override
    public int getRows() {
        return 1 + display.getRows() + 1;
    }

    @Override
    public String getRowText(int row) {
        if (row == 0) {
            return "+" + makeLine('-', display.getColumns()) + "+";
        } else if (row == display.getRows() + 1) {
            return "+" + makeLine('-', display.getColumns()) + "+";
        } else {
            return '|' +display.getRowText(row - 1) + "|";
        }
    }

    private String makeLine(char ch, int count) {
        StringBuffer buffer = new StringBuffer();
        for (int i = 0; i < count; i++) {
            buffer.append(ch);
        }
        return buffer.toString();
    }
}
```

最后Main类进行测试

```java
public class Main {
    public static void main(String[] args) {
        Display b1 = new StringDisplay("Hello,world.");
        Display b2 = new SideBorder(b1, '#');
        Display b3 = new FullBorder(b2);
        b1.show();
        b2.show();
        b3.show();
        Display b4 = new SideBorder(
                new FullBorder(
                        new SideBorder(
                                new FullBorder(
                                        new StringDisplay("你好，世界。")
                                ),
                                '*'
                        )
                ),
                '/'
        );
        b4.show();
    }
}
```

## 登场角色

- Component

  增加功能时的核心角色。本章例子来说，装饰前的蛋糕就是Component角色。Component角色只是定义了蛋糕的接口（API）。

- ConcreteComponent

  实现了Component角色所定义的接口。

- Decorator

  该角色具有与Component角色相同的接口（API），在它内部保存了悲壮是对象-Component角色。

- ConcreteDecorator

  该角色是具体的Decorator角色。

![image-20221016103233590](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221016103233590-1665887554560-3.png)

## 总结

### 接口(API)的透明性

任Decorator嗅式中，装饰边框装饰物具有一致具体细言，在小例程序中，饰边框的Border类是表示被装饰物Display类的子类，这就体现们之间的一致性。也就是说, Border 类( 以及它的子类）与表示破装饰物的Display 类具有相同的接口(API）。

这样，即使被装饰物被边框装饰起来了，接口( API）也不会被隐藏起来。其他类依然可以调用getColumns、getRows、getRowText以及 show方法。这就是接口(API）的“透明性。

在示例程序中，实例b4被装饰了多次，但是接口（API）却没有发生任何变化。

得益于接口（API）的透明性，Decorator模式中也形成了类似于Composite模式中的递归结构。也就是说，装饰边框里面的“被装饰物”实际上又是别的物体的“装饰边框”。就像是剥洋葱时以为洋葱心要出来了，结果却发现还是皮。不过，Decorator模式虽然与Composite 模式一样，都具有递归结构，但是它们的使用目的不同。Decorator模式的主要目的是通过添加装饰物来增加对象的功能。

### 在不改变被装饰物的前提下增加新功能

在Decorator模式中，装饰边框与被装饰物具有相同的接口（API）虽然接口(API)是相同的，但是越装饰，功能则越多。例如，用SideBorder装饰Display后，就可以在字符串的左右内侧加上装饰子行。如果冉用FBorder装饰，那么就可以在子付甲的四周加上边框。此时，我们完全不需要对被装饰的类做任何修改。这样，我们就实现了不修改被装饰的类即可增加功能。

Decorator模式使用了委托。对“装饰边框”提出的要求（调用装饰边框的方法）会被转交（委托）给“被装饰物”去处理。以示例程序来说，就是SideBorder类的getCOlumns 方法调用了display .getColumns（）除此以外，getRows 方法也调用了display .getRows()。

### 可以动态的增加新功能

如果需要一个新功能，只需要重新实现装饰类，就能实现。

### java.io与Decorator模式

下面我们来谈谈 java.io包中的类。java.io包是用于输入输出(Input/Output，简称I/O）的包。这里，我们使用了Decorator模式。

首先，我们可以像下面这样生成一个读取文件的实例。

```java
Reader reader = new FileReader(ldatafile.txt")；
```

然后，我们也可以像下面这样在读取文件时将文件内容放人缓冲区。

```java
Reader reader = new BufferedReader(new FileReader("datafile.txt"))；
```

这样，在生成BufferedReader类的实例时，会指定将文件读取到 FileRaer类的实例中。

再然后，我们也可以像下面这样管理行号。

```java
Reader reader = new LineNumberReader(
New BufferedReader(
New FileReader( "datafile.txt")；
```

下面我们来谈谈 java.io包中的类。java.io包是用于输入输出(Input/Output，简称I/O）的包。这里，我们使用了Decorator模式。

首先，我们可以像下面这样生成一个读取文件的实例。

```java
Reader reader = new FileReader(ldatafile.txt")；
```

然后，我们也可以像下面这样在读取文件时将文件内容放人缓冲区。

```java
Reader reader = new BufferedReader(new FileReader("datafile.txt"))；
```

这样，在生成BufferedReader类的实例时，会指定将文件读取到 FileRaer类的实例中。

再然后，我们也可以像下面这样管理行号。

```java
Reader reader = new LineNumberReader(New BufferedReader(New FileReader( "datafile.txt")))；
```

无论是 LineNumberReader类的构造函数还是 BufferedReader类的构造函数，都可以接收Reader类（的子类）的实例作为参数，因此我们可以像上面那样自由地进行各种组合。我们还可以只管理行号，但不进行缓存处理。

```java
Reader reader = new LineNumberReader(new FileReader("datafile.txt");）；
```

接下来，我们还会管理行号，进行缓存，但是我们不从文件中读取数据，而是从网络中读取数据(下面的代码中省略了细节部分和异常处理）。

```java
java.net.Socket socket = new Socket(hostname, portnumber)；

....

Reader reader = new LineNumberReader(new BufferedReader(new InputstreamReader(socket.getInputstream()))）；
```

这里使用的 InputstreamReader类既接收 getInputstream方法返回的Inputstream类的实例作为构造函数的参数，也提供了Reader类的接口（API）（这属于习过的Adapter模式）。

除了iava.io包以外，我们还在 iavax.swing.b o r der 包中使用了Decorator模式javax.swing.border包为我们提供了可以为界面中的控件添加装饰边框的类。

### 导致增加功能相似且小的类

## 拓展

### 继承-父类与子类的一致性

子类和父类具有一致性。下面我们看一个简单的例子。

```java
class Parent {
    void parentMethod() {
    }
}
```

```java
class Child extends Parent (
    void childMethod() {
    }
}
```

此时, Child 类的实例可以被保存在 Parent类型的变量中，也可以调用从 Parent类中继承的方法。

```java
Parent obj = new Child();
obj .parentMethod();
```

也就是说，可以像操作Parent类的实例一样操作Child类的实例。这是将子类当作父类看待的一个例子。

但是，反过来，如果想将父类当作子类一样操作，则需要先进行类型转换。

```java
Parent obj = new Child();
((Child)obj).childMethod();
```

### 委托-自己和被委托对象的一致性

使用委托让**接口具有透明性**时，自己和被委托对象具有一致性。

下面我们看一个稍微有点生硬的例子。

```java
class Rose {
    Violet obj;
    void method() {
    	obj.method();
    }
}
```

```java
class Violet {
    void method() {
    }
}
```

Rose和Violet都有相同的 method方法。Rose将method方法的处理委托给了 Violet。

这样，会让人有一种好像这两个类有所关联，又好像没有关联的感觉。

要说有什么奇怪的地方，那就是这两个类重然都有method 方法，但是却没有明确地在代码中体现出这个“共通性”如果要明确地表示method方法是共通的. 只需要像一个共通的抽象类 Flower 就可以了(或者是接口)。

至此，大家可能会产生这样的疑问，即Rose类中的obj字段被指定为具体类型 violet 真的好吗？如果指定为抽象类型 Flower 会不会更好呢？……究竟应该怎么做才好呢？其实没有固定答案，需求不同，做法也不同。

## 相关设计模式

- [[02_adapter模式]]
- [[10_Strategy模式]]
