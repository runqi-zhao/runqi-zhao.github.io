---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Facade模式

程序这东西总是会变得越来越大。随着时间的推移，程序中的类会越来越多，而且它们之间相互关联，这会导致程序结构也变得越来越复杂。我们在使用这些类之前，必须先弄清楚它们之间的关系，注意正确的调用顺序。

特别是在调用大型程序进行处理时，我们需要格外注意那些数量庞大的类之间错综复杂的关系。不过与其这么做，不如为这个大型程序准备一个“窗口”。这样，我们就不必单独地关注每个类了，只需简单地对“窗口”提出请求即可。

这个“窗口”就是我们在本章中将要学习的Facade模式。Facade是一个源自法语Facade的单词，它的意思是“建筑物的正面”。

使用Facade模式可以为互相关联在一起的错综复杂的类整理出高层接口(API)。其中的Facade角色可以让系统对外只有一个简单的接口(API)。而且，Facade角色还会考虑到系统内部各个类之间的责任关系和依赖关系，按照正确的顺序调用各个类。

本章中，我们将学习可以为系统提供一个简单窗口的Facade模式。

## 例子

编写一个Web页面。

以用于从邮件地址中获取用户名字的数据库类(Database)，一个用于编写HTML文件的类(HtmlWriter)，以及一个扮演Facade角色并提供高层接口(API)的类。

| 包        | 名字       | 说明                            |
| --------- | ---------- | ------------------------------- |
| pagemaker | Database   | 从邮件地址中获取用户名的类      |
| pagemaker | HtmlWriter | 编写HTML文件的类                |
| pagemaker | PageMaker  | 根据邮件地址编写该用户的Web页面 |
| 无名      | Main       | 测试程序行为的类                |

![image-20221019153132401](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221019153132401-1666164693284-1.png)

Database类可以获取指定数据名所对应的Properties的实例。

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class Database {
    private Database() {
    }

    public static Properties getProperties(String dbname) {
        String filename = dbname + ".txt";
        Properties prop = new Properties();
        try {
            prop.load(new FileInputStream(filename));
        } catch (IOException e) {
            System.out.println("Warning: " + filename + " is not found.");
        }
        return prop;
    }
}
```

数据文件：

```txt
hyuki@hyuki.com=Hiroshi Yuki
hanako@hyuki.co=Hanako Stato
tmoura@yuki.com=Tomura
```

HtmlWriter类用于编写简单的Web页面。

```java
public class HtmlWriter {
    private Writer writer;

    public HtmlWriter(Writer writer) {
        this.writer = writer;
    }

    public void title(String title) throws IOException {
        writer.write("<html>");
        writer.write("<head>");
        writer.write("<title>" + title + "</title>");
        writer.write("</head>");
        writer.write("<body>\n");
        writer.write("<h1>" + title + "</h1>\n");
    }

    public void paragraph(String msg) throws IOException {
        writer.write("<p>" + msg + "</p>\n");
    }

    public void link(String href, String caption) throws IOException {
        paragraph("<a href=\"" + href + "\">" + caption + "</a>");
    }

    public void mailto(String mailaddr, String username) throws IOException {
        link("mailto:" + mailaddr, username);
    }

    public void close() throws IOException {
        writer.write("</body>");
        writer.write("</html>\n");
        writer.close();
    }
}
```

PageMaker类使用Database类和HtmlWriter类来生成指定用户的web页面。

PageMaker类一手包办了调用HtmlWriter类的方法这一工作。对外部，他只他过那个了makeWelcomePage接口。

```java
public class PageMaker {
    private PageMaker() {
    }

    public static void makeWelcomePage(String mailaddr, String filename) {
        try {
            Properties mailprop = Database.getProperties("maildata");
            String username = mailprop.getProperty(mailaddr);
            HtmlWriter writer;
            writer = new HtmlWriter(new FileWriter(filename));
            writer.title("Welcome to " + username + "'s page!");
            writer.paragraph(username + "のページへようこそ。");
            writer.paragraph("メール待っていますね。");
            writer.mailto(mailaddr, username);
            writer.close();
            System.out.println(filename + " is created for " + mailaddr + " (" + username + ")");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

Main方法进行测试

```java
public class Main {
    public static void main(String[] args) {
        PageMaker.makeWelcomePage("hyuki@hyuki.com", "welcome.html");
    }
}
```

## 登场角色

### Facade

代表构成系统的许多其他角色“简单窗口”。Facade角色像系统外部提供高层接口(API)。

### 构成系统的许多其他角色

这些角色各自完成自己的工作，它们并不知道Facade角色。Facade角色调用其他角色进行工作，但是其他角色不会调用Facade角色。在示例程序中，由Database类和HtmlWriter类扮演此角色。

### Client(请求者)

Client角色负责调用Facade角色（Client角色并不包含在Facade模式中)。在示例程序中，由Main类扮演此角色。

![image-20221019154305311](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221019154305311-1666165386491-3.png)

## 总结

### Facade角色到底做了什么工作

Facade模式可以让复杂的东西看起来简单。那么，这里说到的“复杂的东西”到底是什么呢？其实就是在后台工作的这些类之间的关系和它们的使用方法。使用Facade模式可以让我们不必在意这些复杂的东西。

这里的重点是接口(API)变少了。程序中如果有很多类和方法，我们在决定到底应该使用哪个类或是方法时就很容易迷茫。有时，类和方法的调用顺序也很容易弄错，必须格外注意。因此，如果有一个能够使接口（API)变少的Facade角色是一件多么美好的事情啊。

接口(API)变少了还意味着程序与外部的关联关系弱化了，这样更容易使我们的包（类的集合)作为组件被复用。

在设计类时，我们还需要考虑将哪些方法的可见性设为public。如果公开的方法过多，会导致类的内部的修改变得困难。字段也是一样的，如果不小心将某个字段公开出去了，那么其他类可能会读取或是修改这个字段，导致难以修改该类。

与设计类一样，在设计包时，需要考虑类的可见性。如果让外部（包的外部）看到了类，包内部代码的修改就会变得困难。

### 递归地使用Facade模式

既然上面我们从设计类谈到了设计包的问题，下面就让我们把思考范围再扩大一些。

假设现在有几个持有Facade角色的类的集合。那么，我们可以通过整合这几个集合来引人新的Facade角色。也就是说，我们可以递归地使用Facade模式。

在超大系统中，往往都含有非常多的类和包。如果我们在每个关键的地方都使用Facade模式，那么系统的维护就会变得轻松很多。

## 相关设计模式

- [[08_Abstract_Fatctory模式]]
- [[05_Singleton模式]]
- [[16_Mediator模式]]

