---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Abstract Factory模式

Abstract Factory模式的工作是将“抽象零件”组装成“抽象产品”。

面向对象编程中“抽象”这个词的具体含义是“补考看v具体怎样实现，而是仅关注接口（API）”的状态，例如，抽象方法并不定义放到发的具体实现，而是仅仅只确定了放啊的名字和签名（参数的类型和个数）。

我们并不关心零件的具体实现，而是只关心接口（API）。我们仅使用该接口（API）将零件组装成为产品。

在Tempate Method模式和Builder模式中，子类这一层负责方法的具体实现。在Abstract Factory模式中也是一样的。在子类这一层中有具体的工厂，它负责将具体的零件组常称为具体的产品。

## 例子

将带有层次关系的集合制作成HTML文件。

在示例程序中，类被划分为以下3个包：

- factory包：包含抽象工厂、零件、产品的包
- 无名包：包含Main的包
- listfactory包：包含具体工厂、零件、产品的包（这里使用`<ul>`标签输出为HTML文件）

| 包          | 名字        | 说明                                                 |
| ----------- | ----------- | ---------------------------------------------------- |
| factory     | Factory     | 表示抽象工厂的类（制作Link、Tray、Page）             |
| factory     | Item        | 方便统一处理Link和Tray的类                           |
| factory     | Link        | 抽象零件：表示HTML的链接的类                         |
| factory     | Tray        | 抽象零件：表示含有Link和Tray的类                     |
| factory     | Page        | 抽象零件：表示HTML页面的类                           |
| 无名        | Main        | 测试成行为的类                                       |
| listfactory | ListFactory | 表示具体工厂的类（制作ListLink、ListTray、ListPage） |
| listfactory | ListLink    | 具体零件：表示含有HTML的连接类                       |
| listfactory | ListTray    | 具体零件：表示含有Link和Tray的类                     |
| listfactory | ListPage    | 具体零件：表示HTML页面的类                           |

![image-20221012185819442](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221012185819442-1665572300383-1.png)

![image-20221012185838524](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221012185838524-1665572319928-3.png)

编译方法如下：

```cmake
javac Main.java listfactory/ListFactory.java
```

下面看具体的代码实现：

Item类是Link类和Tray类的父类，这样，Link类和Tray类就具有可替换性了。

caption字段表示项目中的“标题”

makeHTML是抽象方法，需要子类去实现。该方法会返回HTML文件的内容（需要子类去实现）

```java
public abstract class Item {
    protected String caption;

    public Item(String caption) {
        this.caption = caption;
    }
    public abstract String makeHTML();
}

```

Link类是抽象的表示HTML的超链接的类。

url字段中保存的是超链接所指向的地址，由于Link类中没有实现父类（Item类）的抽象方法（makeHTML），因此它也是抽象类

```java
public abstract class Link extends Item {
    protected String url;

    public Link(String caption, String url) {
        super(caption);
        this.url = url;
    }
}
```

Tray类表示将一个含有多个Link类和Tray类的容器

```java
public abstract class Tray extends Item {
    protected ArrayList<Item> tray = new ArrayList<>();
    public Tray(String caption) {
        super(caption);
    }

    public void add(Item item) {
        tray.add(item);
    }
}
```

Page类是抽象地表示HTML页面的类。如果将Link和Tray比喻成抽象的“零件”，那么Page类就是抽象的“产品”。title和author分别是表示页面标题和页面作者的字段。作者名字通过参数传递给Page类的构造函数。

可以使用add方法向页面中增加Item(即工ink或Tray)。增加的Item将会在页面中显示出来。output方法首先根据页面标题确定文件名，接着调用makeHTML方法将自身保存的HTML内容写入到文件中。

其中，我们可以去掉如下语句(1)中的this,将其写为如下语句(2)那样。
writer.write(this.makeHTML());	(1)
writer.write(makeHTML());	(2)
为了强调调用的是Page类自己的makeHTML方法，我们显式地加上了this。这里调用的makeHTML方法是一个抽象方法。output方法是一个简单的Template Method模式的方法。

```java
public abstract class Page {
    //这里不能是Item的子类，如果是的话，则将会Tray中能添加Page，违反了html的规范
    protected String title;
    protected String author;
    protected ArrayList<Item> content = new ArrayList<>();

    public Page(String title, String author) {
        this.title = title;
        this.author = author;
    }

    public void add(Item item) {
        content.add(item);
    }

    public void output() {
        try {
            String filename = title + ".html";
            Writer writer = new FileWriter(filename);
            writer.write(this.makeHTML());
            writer.close();
            System.out.println(filename + "编写完成。");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public abstract String makeHTML();
}
```

Factory类是抽象工厂，其中的getFactory方法可以根据指定的类名生成具体工厂的实例。

getFactory方法通过**调用Class类的forName方法来动态地读取类信息，接着使用newInstance方法生成该类地实例**，并将其作为返回值返回给调用者。

Class类属于java.lang包，是用来表示类的类。Class类包含于Java的标准类库中。forName是java.lang.Class的类方法（静态方法），newInstance则是java.lang.Class的实例方法。（关于这里，运用到了反射地机制，这里的话请看我的另一篇博文总结，很重要一个点）。

createLink、createTray、createPage等方法是用于在抽象工厂中生成零件和产品的方法。这些方法都是抽象方法，具体的实现被交给了Factory类的子类。不过，这里确定了方法的名字和签名。

```java
public abstract class Factory {
    public static Factory getFactory(String classname) {
        Factory factory = null;
        try {
            factory = (Factory) Class.forName(classname).newInstance();
        } catch (ClassNotFoundException e ) {
            System.err.println("没有找到" + classname + "类。");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return factory;
    }

    public abstract Link createLink(String caption, String url);
    public abstract Tray createTray(String caption);
    public abstract Page createPage(String title, String author);
}
```

在理解了抽象的零件、产品、工厂的代码后，我们来看看Main类的代码。Main类使用抽象工厂生产零件并将零件组装成产品。Main类中只引入了factory包，从这一点可以看出，该类并没有使用任何具体零件、产品和工厂。

具体工厂的类名是通过命令行来指定的。例如，如果要使用listfactory包中的ListFactory类，可以在命令行中输人以下命令。

```
java Main listfactory.ListFactory
```

Main类会使用getFactory方法生成该参数(arg[0])对应的工厂，并将其保存在factory变量中。

之后，Main类会使用factory生成Link和Tray,然后将Link和Tray都放入Tray中，最后生成Page并将生成结果输出至文件。

```java
public class Main {
    public static void main(String[] args) {
        if (args.length != 1) {
            System.out.println("Usage : java Main.class.name.of.ConcreteFactory");
            System.out.println("Example 1 : java Main listFactory");
            System.out.println("Example 2 : java Main tablefactory.TableFactory");
            System.exit(0);
        }
        Factory factory  = Factory.getFactory(args[0]);

        Link people = factory.createLink("人民日报", "https://www.people.com.cn/");
        Link gmw = factory.createLink("光明日报", "https://www.gmw.cn/");
        Link us_yahoo = factory.createLink("Yahoo!","https://www.yahoo.cn/");
        Link excite = factory.createLink("Excite", "https://www.excite.com/");
        Link google = factory.createLink("google", "https://www.google.com/");

        Tray traynews = factory.createTray("日报");
        traynews.add(people);
        traynews.add(gmw);

        Tray trayyahoo = factory.createTray("Yahoo!");
        trayyahoo.add(us_yahoo);

        Tray traySearch = factory.createTray("检索引擎");
        traySearch.add(excite);
        traySearch.add(google);

        Page page = factory.createPage("LinkPage", "杨文轩");
        page.add(traynews);
        page.add(traySearch);
        page.output();
    }
}
```

之前都是抽象类，现在将目光转向具体类，来看listfactory包中的工厂-ListFactory类。

ListFactory类实现了Factory类的createLink方法、createTray方法以及createPage方法。当然，各个方法的内部只是分别简单地new出ListLink实例、ListTray实例以及ListPag类地实例（根据实际要求，这里可能需要使用Prototype模式来进行clone）。

```java
public class ListFactory extends Factory {
    @Override
    public Link createLink(String caption, String url) {
        return new ListLink(caption, url);
    }

    @Override
    public Tray createTray(String caption) {
        return new ListTray(caption);
    }

    @Override
    public Page createPage(String title, String author) {
        return new ListPage(title, author);
    }
}

```

ListLink是Link地子类，事先声明的makeHTML方法。

```java
public class ListLink extends Link {

    public  ListLink(String caption, String url) {
        super(caption, url);
    }
    @Override
    public String makeHTML() {
        return "<li><a href=\"" + url + "\">" + caption + "</a></li>\n";
    }
}
```

ListTray类（代码清单8-9）是Tray类的子类。这里我们重点看一下makeHTML方法是如何实现的。tray字段中保存了所有需要以HTML格式输出的Item,而负责将它们以HTML格式输出的就是makeHTML方法了。那么该方法究竟是如何实现的呢？

makeHTML方法首先使用`<li>`标签输出标题(caption),接着使用`<ul>`和`<li>`标签输出每个Item。输出的结果先暂时保存在StringBuffer中，最后再通过toString方法将输出结果转换为String类型并返回给调用者。

那么，每个Item又是如何输出为HTML格式的呢？当然就是调用每个Item的makeHTML方法了。请注意，这里并不关心变量item中保存的实例究竞是ListLink的实例还是ListTray的实例，只是简单地调用了item.makeHTM()语句而已。这里**不能使用switch语句或if语句去判断变量item中保存的实例的类型，否则就是非面向对象编程了**。变量item是Item类型的，而Item类又声明了makeHTML方法，而且ListLink类和ListTray类都是Item类的子类，因此可以放心地调用。之后item会帮我们进行处理。至于item究竟进行了什么样的处理只有item的实例（对象）才知道。这就是面向对象的优点。

这里使用的java.util.Iterator类与我们在Iterator模式一章中所学习的迭代器在功能上是相同的，不过它是Java类库中自带的。为了从java,util.ArrayList类中得到java.util.Iterator,我们调用iterator方法。

```java
public class ListTray extends Tray {

    public ListTray(String caption) {
        super(caption);
    }

    @Override
    public String makeHTML() {
        StringBuffer buffer = new StringBuffer();
        buffer.append("<li>\n");
        buffer.append(caption + "\n");
        buffer.append("<ul>\n");
        Iterator<Item> it = tray.iterator();
        while (it.hasNext()) {
            Item item = (Item) it.next();
            buffer.append(item.makeHTML());
        }
        buffer.append("</ul>\n");
        buffer.append("</li>\n");
        return buffer.toString();
    }
}
```

### 增加另一个例子

当我们只有一个具体工厂的时候，完全没有必要使用“抽象类”与“具体类”如此划分接下来，我们增加另一个工厂类，使用表格将里面具体内容展示出来。

| 包           | 名字         | 说明                                                    |
| ------------ | ------------ | ------------------------------------------------------- |
| tablefactory | TableFactory | 表示具体工厂的类（制作TableLink、TableTray、TablePage） |
| tablefactory | TableLink    | 具体零件：表示HTML的超链接类                            |
| tablefactory | TableTray    | 具体零件：表示含有Link和Tray的类                        |
| tablefactory | TablePage    | 具体产品：表示HTML页面的类                              |

实现的话其实和上面工厂差不多，这里直接写代码

```java
public class TableFactory extends Factory {
    @Override
    public Link createLink(String caption, String url) {
        return new TableLink(caption, url);
    }

    @Override
    public Tray createTray(String caption) {
        return new TableTray(caption);
    }

    @Override
    public Page createPage(String title, String author) {
        return new TablePage(title, author);
    }
}
```

```java
public class TableLink extends Link {
    //这里仅仅调用了父类构造函数，不需要任何处理，为什么还要实现呢？
    //答：这是因为在Java中无法继承构造函数。
    //即使在父类中有Link(String caption,String url)构造函数，如果在ListLink类中不定义ListLink(String caption,String url)构造函数，就无法像下面这样生成实例，还会在编译代码时出错。
    public TableLink(String caption, String url) {
        super(caption, url);
    }

    @Override
    public String makeHTML() {
        return null;
    }
}
```

```java
public class TableTray extends Tray {
    public TableTray(String caption) {
        super(caption);
    }

    @Override
    public String makeHTML() {
        return null;
    }
}
```

```java
public class TablePage extends Page {
    public TablePage(String title, String author) {
        super(title, author);
    }

    @Override
    public String makeHTML() {
        return null;
    }
}
```

## 登场角色

- AbstractProduct（抽象产品）

  负责定义AbstractFactory角色所生成的抽象零件和抽象产品的接口（API）。

- AbstractFactory（抽象工厂）

  负责定义用于生成抽象产品的接口（API）。

- Client（委托者）

  进会调用AbstractFactory角色和AbstractProduct角色的接口（API）来进行工作，对于具体的零件、产品和工厂一无所知。

  ![image-20221012194000597](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221012194000597-1665574804099-5.png)

- ConcreteProduct（具体产品）

  负责实现AbstractProduct角色的接口（API）

- ConcreteFactory（具体工厂）

  负责实现AbstractFactory角色的接口（API）

## 总结

### 易于增加具体的工厂

在Abstract Factory模式中增加具体的工厂是非常容易的。这里说的“容易”指的是需要编写哪些类和需要实现哪些方法都非常清楚。

假设现在我们要在示例程序中增加新的具体工厂，那么需要做的就是编写Factory、工ink、Tray、Page这4个类的子类，并实现它们定义的抽象方法。也就是说将factory包中的抽象部分全部具体化即可

这样一来，无论要增加多少个具体工厂（或是要修改具体工厂的Bug),都无需修改抽象工厂和Main部分。

### 难以增加新的零件

请试想一下要在Abstract Factory模式中增加新的零件时应当如何做。例如，我们要在factory包中增加一个表示图像的Picture零件。这时，我们必须要对所有的具体工厂进行相应的修改才行。例如，在listfactory包中，我们必须要做以下修改。

- 在ListFactory中加入createpicture方法
- 新增ListPicture类

已经编写完成的具体工厂越多，修改的工作量就会越大。

## 补充知识点：各种生成实例的方法介绍

- new

  一般我们使用Java关键字new生成实例.

  可以像下面这样生成Something类的实例并将其保存在obj变量中。

  ```java
  Something obj new Something();
  ```

  这时，类名（此处的Something)会出现在代码中（即为请耦合关系）。

- clone

  我们也可以使用在[Prototype模式](https://www.runqi.live/docs/skill/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/Prototype%E6%A8%A1%E5%BC%8F#%E4%BE%8B%E5%AD%90)中学习过的clone方法，根据现有的实例复制出一个新的实例。

  我们可以像下面这样根据自身来复制出新的实例（不过不会调用构造函数）。

- newInstance

  使用本章中学习过的java.lang.class类的newInstance方法可以通过class类的实例生成出c1ass类所表示的类②的实例（会调用无参构造函数）。

  在本章的示例程序中，我们已经展示过如何使用newInstance了。下面我们再看一个例子。假设我们现在已经有了Something类的实例someobj,通过下面的表达式可以生成另外一个Something类的实例。

  ```java
  someobj.getclass().newInstance()
  ```

  实际上，调用newInstance方法可能会导致抛出InstantiationException异常或是IllegalAccessException异常，因此需要将其置于try.catch语句块中或是用throws关键字指定调用newInstance方法的方法可能会抛出的异常。

这里请看我另一篇博客-深拷贝、浅拷贝，以及另一篇文章-反射机制。

## 相关设计模式

- [[07_Builder模式]]
- [[04_Factory_Method模式]]
- [[11_Composite模式]]
- [[05_Singleton模式]]
